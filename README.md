# Callback

-	[Información General](#información-general)

-	[Custom Callback](#custom-callback)
        
-	[Tipos de Callback](#tipos-de-callback)

	-	[Data](#data)
	
	-	[Service](#service)
        
        
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
