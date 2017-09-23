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


Callback Medium
---------------

El medio define la manera en cómo se reenvía el evento

## Email

Aquí se necesita configurar un email válido, asunto y el cuerpo del mensaje.
El asunto y el cuerpo del mensaje puede contener arbitrariamente texto con las variables debidamente definidas con llaves.


## Http

Aquí se necesita implementar un servicio RESTful o webFacing. Existen dos tipos de callback 

#### Simple

Cada mensaje es reenviado directamente en una petición HTTP simple. Se puede usar GET, POST, PUT, aunque POST es el mas recomendado

##### Variables 

Dependiendo del callback, distintas variables están disponibles. La lista de variables disponibles está desplegada arriba del campo de URL. Estas variables pueden ser usada en 3 lugares.

-    URL: Como direcccion de variables o parametros de petición.
-    Headers: Como valores del header. Las variables no pueden ser usadas en un key de header ya que el formato está 		estandarizado
-    Body: Si se elige el método POST o PUT, se puede definir un template que contenga las variables. Estas variables son 	reemplazadas con el valor correspondiente

##### Headers 

Se pueden definir headers personalizados en los callbacks. Por seguridad, deshabilitamos todos los headers estandarizados menos 'Authorization'. Este header te permite usar otro método de autenticación que solo Basic. No se permite poner el mismo header duplicado. Asi como se puede poner la información del usuario en la URL en la forma de http://login:password@yourdomain.com, se cauteloso al poner un header de autorización.

