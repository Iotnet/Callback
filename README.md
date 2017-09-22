# Callback
==========

-	    [Información general](#información+general)

-	    [Custom Callback](#custom+callback)

        -	    [Tipos de Callback](#tipos+de+callback)

                 -	    [Data](#data)
        
        
## Información General

El backend puede reenviar automaticamente eventos utilizando el sistema de "Calback".
Un 'Callback' es una petición http que contiene la información del dispositivo, además de otras variables que son enviadas a una plataforma y/o servidor.

El callback se dispara cuando un mensaje de un dispositivo es recibido, cuando una locación ha sido estiamada o cuando se ha detectado una perdida de comunicación con el dispositivo. 

La configuración se realiza en la pagina  "Device Type" dentro del backend de Sigfox
Para estar seguro que tu aplicacion/servidor puede recibir callbacks, debes de asegurarte la dirección 185.110.97.0/24 esta permitida. Ésta esta en notación CIDIR.

*La dirección IP puede cambiar. Cualquier cambio se hará saber inmediatamente.*

## Custom Callback

Existen 3 tipos de callback y se cuenta con un set de variables disponibles por cada tipo de callback.
Estas variables son reemplazadas por el valor correspondiente cuando el callback es llamado.
Cuando se recibe un callback, el cliente debe regresar un código HTTP 2xx dentro de los 10 segundos siguientes. Si el cliente falla en procesar el callback durante este tiempo, automaticamente se realiza otro intento 1 minuto después. 


### Tipos de Callback

Cada tipo de callback comparte un set de variables en común. 
        -       Time (int): the event timestamp (in seconds since the Unix Epoch)

#### Data


