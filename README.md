# Callback

-	[Información General](#información-general)

-	[Custom Callback](#custom-callback)
        
-	[Tipos de Callback](#tipos-de-callback)

	-	[Data](#data)
	
	-	[Service](#service)
	
	-	[Error](#error)
	
-	[Callback Medium](#callback-medium)

	-	[EMAIL](#email)
	
	-	[HTTP](#http)
	
-	[Downlink messages](#downlink-messages)

-	[Ejemplo](#ejemplo)
        
		
Información General
-------------------

El backend puede reenviar automaticamente eventos utilizando el sistema de "Calback".
Un 'Callback' es una petición http que contiene la información del dispositivo, además de otras variables que son enviadas a una plataforma y/o servidor.

El callback se dispara cuando un mensaje de un dispositivo es recibido, cuando una locación ha sido estimada o cuando se ha detectado una perdida de comunicación con el dispositivo. 

La configuración se realiza en la pagina  "Device Type" dentro del backend de Sigfox
Para estar seguro de que tu aplicacion/servidor puede recibir callbacks, debes de asegurarte la dirección 185.110.97.0/24 está permitida. Ésta esta en notación CIDIR.

*La dirección IP puede cambiar. Cualquier cambio se hará saber inmediatamente.*

Custom Callback
---------------

Existen 3 tipos de callback y se cuenta con un set de variables disponibles por cada tipo de callback.
Estas variables son reemplazadas por el valor correspondiente cuando el callback es llamado.
Cuando se recibe un callback, el cliente debe regresar un código HTTP 2xx dentro de los 10 segundos siguientes. Si el cliente falla en procesar el callback durante este tiempo, automaticamente se realiza otro intento 1 minuto después. 

Tipos de Callback
-----------------

Cada tipo de callback comparte una variable en común. 

-	Time (int): Evento de marca de tiempo (En segundos - Unix Epoch)


## Data

En este tipo de callback el usuario puede configurar variable personalizadas que serán reemplazadas por el valor parseado. Entonces se podrán ocupar estas variables en el callback

*Para obtener mas información sobre el formato de configuración revisar la ayuda en linea de Sigfox*

Este tipo de callback define la entrada de un mensaje proveniente de un dispositivo. Las variables disponibles son:ssi
-    device (string): Identificador del dispositivo (device ID) en hexadecimal hasta 8 caracteres = 4 bytes
-    duplicate (bool): <verdadero> si el mensaje es un duplicado, lo que significa que el backend ha procesado el mensaje de
        otra radio-base, <falso> de otra manera
-    snr (float): La relación señal a ruido (en dB - flotante con máximo 2 dígitos de fracción)
-    rssi (float): El RSSI (en dBm - flotante con máximo 2 digitos de fracción) si no hay valores de retorno entonces el valor 			será null
-    avgSnr (float): Valor promedio de la relación señal a ruido calculada de los últimos 25 mensajes (en dB -  Flotante con 			máximo dos dígitos de fracción) o N/A. El dispositivo tiene que enviar al menos 15 mensajes
-    station (string): EL identificador de la radio-base (En hexadecimal - 4 caracteres - 2 bytes)
-    data (string): Los datos del usuario (En hexadecimal) 
-    lat (float): Latitud redondeada al entero más cercano, de la radio base más cercana que recibió el mensaje
-    lng (float): Longitud, redondeada al entero más cercano, de la radio base más cercana que recibió el mensaje
-    seqNumber (int): Número de secuencia del mensaje si esta disponible
	
	![data](https://github.com/Iotnet/Callback/blob/master/images/data.png)

        
#### UPLINK

Este subtipo no define otra variable adicional. Sólo define que el callback será de subida es decir, hacia la nube.

#### BIDIR

-    ack (bool): Verdadero si el mensaje necesita ser acknowledged, falso si no.

El cliente puede decidir si mandar o no respuesta hacia el dispositivo. Para este caso hay dos maneras de hacerlo:

-    Responder al callback con el código HTTP NO_CONTENT (204)
-    Responder con un mensaje json que contenga el campo *noData*:
                
			{ "0CB3":
            		{	
            			"noData" : true
            		}
			}

## Service

Este tipo de callback define la recepción de un mensaje operativo de un dispositivo. Variables disponibles son:

-    device (string): Identificador del dispositivo (device ID) en hexadecimal hasta 8 caracteres = 4 bytes
-    duplicate (bool): <verdadero> si el mensaje es un duplicado, lo que significa que el backend ha procesado el mensaje de 		otra radio base, <falso> de otra manera
-    avgSnr (float): Valor promedio de la relación señal a ruido calculada de los últimos 25 mensajes (en dB -  Flotante con 		máximo dos dígitos de fracción) o N/A. El dispositivo tiene que enviar al menos 15 mensajes
-    station (string): El identificador de la radio-base (En hexadecimal - 4 caracteres - 2 bytes)
-    lat (float): Latitud redondeada al entero más cercano, de la radio base más cercana que recibió el mensaje
-    lng (float): Longitud, redondeada al entero más cercano, de la radio base más cercana que recibió el mensaje
-    signal (float): La relación señal a ruido (En dB - Flotante con máximo dos dígitos de fracción)	

	![service](https://github.com/Iotnet/Callback/blob/master/images/service.png)


Para este tipo se encuentran disponibles las siguientes opciones de configuración:

#### Status

-    batt (float): El voltaje de la batería (en Volts - Flotante con máximo dos dígitos de fracción)
-    temp (float): Temperatura del dispositivo (en ºC - Flotante con máximo dos dígitos de fracción)
-    seqNumber (int): Número de secuencia del mensaje si está disponible


#### Acknowledge

-    infoCode (int): Es el código de status del downlink:

	-    0 (ACKED) recibi: La estación emitió la respuesta
	-    1 (NO_ANSWER): El cliente no dió ninguna respuesta
	-    2 (INVALID_PAYLOAD): La información enviada al dispositivo es inválida
	-    3 (OVERRUN_ERROR): El dispositivo excedió su cuota de mensajes downlink 
	-    4 (NETWORK_ERROR): No fué posible transmitir la respuesta
	-    5 (PENDING): Hay un código que está pendiente enviar al dispositivo
	-    6 (NO_DOWNLINK_CONTRACT): El dispositivo espera respuesta pero su BSS no acepta downlink
	-    7 (TOO_MANY_REQUESTS): El dispositivo espera una respuesta antes de que el tiempo de respuesta expire
	-    8 (INVALID_CONFIGURTION): El device type está configurado para obtener información del callback pero ningún
	       callback BIDIR ha sido definido
-    infoMessage (string): Mensaje asociado al código
-    downlinkAck (bool): Verdadero si la estación permite la transmisión, falso si no.
-    downlinkOverusage (bool): Verdadero si el dispositivo excede su cuota diaria, falso si no.

#### Repeater

-    batt (float): El voltaje de la batería (en Volts - Flotante con máximo dos dígitos de fracción)
-    whitelistDevices (int): Numero de dispositivos en la lista de aceptados. Si es 0, todos los mensajes son repetidos
-    repeatedMsg (int): Número de mensajes repetidos desde la última transmisión de datos
-    unwantedDevices (int): Número de dispositivos únicos  que no están en la lista de permitidos desde la última transmisión
	de datos. Si es 15 significa que el numero es mayor que 14
-    unwantedMsg (int): El rango de la suma de mensajes recibidos desde la última transmisión de datos.
-    wrongMsg (int): El rango del total de mensajes equivocados recibidos desde última transmisión.
-    overRegulmsg (int): Rango de mensajes no repetidos debido a.

## Error

Este callback permite saber la pérdida de comunicación del dispositivo con el backend. Las variables personalizadas son:

-    device (string): Identificador del dispositivo (device ID) en hexadecimal hasta 8 caracteres = 4 bytes
-    info (string): Información del error, en caso de pérdida de comunicación, contiene la fecha del último mensaje recibido
-    severity (string): <ERROR> cuando es un problema del dispositivo, <WARN> cuando la conexión está experimentando problemas 		que pudieran causar pérdidas de mensajes.

	![error](https://github.com/Iotnet/Callback/blob/master/images/error.png)


Callback Medium
---------------

El medio define la manera en cómo se reenvía el evento

## Email

Aquí se necesita configurar un email válido, asunto y el cuerpo del mensaje.
El asunto y el cuerpo del mensaje puede contener arbitrariamente texto con las variables debidamente definidas con llaves.


## Http

Aquí se necesita implementar un servicio RESTful o webFacing. Existen dos tipos de callback 

### Simple

Cada mensaje es reenviado directamente en una petición HTTP simple. Se puede usar GET, POST, PUT, aunque POST es el mas recomendado

##### Variables 

Dependiendo del callback, distintas variables están disponibles. La lista de variables disponibles está desplegada arriba del campo de URL. Estas variables pueden ser usada en 3 lugares.

-    URL: Como direcccion de variables o parametros de petición.
-    Headers: Como valores del header. Las variables no pueden ser usadas en un key de header ya que el formato está 		estandarizado
-    Body: Si se elige el método POST o PUT, se puede definir un template que contenga las variables. Estas variables son 	reemplazadas con el valor correspondiente

##### Headers 

Se pueden definir headers personalizados en los callbacks. Por seguridad, deshabilitamos todos los headers estandarizados menos 'Authorization'. Este header te permite usar otro método de autenticación que solo Basic. No se permite poner el mismo header duplicado. Asi como se puede poner la información del usuario en la URL en la forma de http://login:password@yourdomain.com, se cauteloso al poner un header de autorización.

##### GET Method
Las variables son reemplazadas automáticamente en la cadena de consulta 

	GET http://hostname/path?id={device}&time={time}&key1={data}&key2={signal}

##### POST Method

Los métodos POST o PUT permiten configurar el content-type y el cuerpo de la petición. Se puede elegir entre 3 tipos de content-type:

-	**application/x-www-form-unlencoded**

Éste es el content-type predefinido para los métodos POST y PUT. Cuando se usa este content-type puedes configurar el body en un formato enconded:

    device={device}&data={data}&time={time}
	
Si el formato no se respeta sera denegado.

**Notar que si se pone algúnas variables en la cadena de la URL, esos parametros serán adjuntados ya sea si esta vacía o no.*

	Ejemplo:

	POST http://hostname/path?id={device}&time={time}&key1={data}&key2={signal}

con el body de la siguiente manera:
device={device}&data={data}&time={time}

resultará en el siguiente body:

	id={device}&time={time}&key1={data}&key2={signal}&device={device}&data={data}&time={time}

-	**application/json**

De igual manera se puede configurar un objeto JSON en el body

	{
        "device" : "{device}",
        "data" : "{data}",
        "time" : {time}
    }
	
Cuando se elige este content-type, el objeto JSON es validado. Si no se respeta el formato JSON, será rechazado
*Si se requiere de mayor información acerca de la notación JSON, obtener ayuda en linea*

-	**text/plain**

Este content-type es libre. No se realiza ninguna validación (Excepto caracteres omitidos) 
Ejemplo:

	device => {device}
    data => {data}
    time => {time}
	

### Batch

Los mensajes son reunidos por callback, cada linea representa un mensaje, luego son enviados en un batch usando una petición HTTP cada segundo. Esto evita una sobrecarga que el servidor no pueda soportar. Como el payload contiene varios mensajes, solo el método POST es soportado. Cuando se usan los **batch callbacks** con la opción de duplicado activa, no hay garantía de que los mensajes duplicados de un mensaje sean agrupados en un mismo batch. Todo depende del momento exacto de procesamiento de cada uno de esos duplicados 

	POST http://hostname/path?batch={batch} where batch={device};{data};{signal};...
	

Downlink messages
-----------------

Cuando un mensaje necesita ser aceptado, el callback seleccionado por el downlink data necesita enviar información en la respuesta, este debe contener los 8 bytes de información que serán enviados al dispositivo preguntando por el reconocimiento.
La información estará en formato JSON, y debe ser estructurado como sigue:

	{
    	"device_id" : { "downlinkData" : "deadbeefcafebabe"}
	}
	
Con *device_id* reemplazado por el correspondiente device_id en formato hexadecimal y hasta 8 digitos. La información de downlink deberá ser de 8 bytes en formato hexadecimal.

Con lo **batch callbacks**, multiples dispositivos necesitan un acknowledge. La respuesta deberá contener la información de todos estos dispositivos. La respuesta tiene la siguiente forma:

	{
    	"device1_id" : { "downlinkData" : "deadbeefcafebabe"},
    	"device2_id" : { "downlinkData" : "bebebabab0b0b1b1"}
	}


Ejemplo
-------

En este ejemplo se realizará un ejemplo de callback utilizando como plataforma de visualización de datos Ubidos y un botón de emergencia

Primero nos dirigiremos a ![Ubidots](https://ubidots.com/) y nos registraremos. 

La pagina nos redirigirá automaticamente al dashboard. Lo que necesitamos de nuestra cuenta son las API Credentials, en específico el token. Este lo encontramos en el lado superior izquierdo dando click en nuestro nombre de usuario y despues en 'API Credentials'.

![Ubidots1](https://github.com/Iotnet/Callback/blob/master/images/ubidots1.png)

Nos aparecerá una ventana en la parte superior. Copiaremos y pegaremos únicamente el **Token** en algún lado para ocuparlo mas adelante.

![Ubidots2](https://github.com/Iotnet/Callback/blob/master/images/ubidots2.png)


Como se describe en esta guía, los callbacks se realizan desde la pagina "Device Type" en el backend de Sigfox. Por lo tanto nos dirigimos ahí. Y buscamos por nuestro Device Type.

![DeviceType](https://github.com/Iotnet/Callback/blob/master/images/devicetype.png)

Dentro del nuestro device type encontraremos en el lado derecho un menu, buscaremos la opción "Callbacks" y daremos clicks.

![DeviceType1](https://github.com/Iotnet/Callback/blob/master/images/devicetype1.png)

Y del lado derecho daremos click en "New"

Notaremos que hay distintos tipos de Callback como AWS IoT, AWS Kinesis, Microsoft Azure etc. Estos callbacks ya esta´n
 predefinidos para usar en esas plataforma. En nuestro caso Ubidots no está en la lista por lo que tendremos que realizar un Custom Callback.
 
 Este tipo de Callback permite enviar los datos que llegan al cloud de Sigfox a tu propio servidor/web application. 

![Device Type2](https://github.com/Iotnet/Callback/blob/master/images/devicetype2.png)

Configuraremos nuestro callback de la siguiente manera:

![ubidots3](https://github.com/Iotnet/Callback/blob/master/images/ubidots3.png)


Donde: 

CHANNEL: URL

URL PATTERN: http://things.ubidots.com/api/v1.6/devices/{device}/?token={Put_Your_Ubidots_TOKEN_here}

Use HTTP Method : POST

BODY: 	{"data" : "{data}"}

