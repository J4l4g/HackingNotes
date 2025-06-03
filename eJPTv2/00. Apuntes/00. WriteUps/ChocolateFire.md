
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 172.17.0.2 -oN ScanPorts.txt`

Encontramos que el puerto 9090 esta levantado, en este esta corriendo un openfire.
Openfire es un servicio de mensajería instantánea. Buscamos si hay algún CVE en Google y si lo encontramos, nos metemos en metasploit y buscamos el servicio que queremos explotar. Buscamos que correspondan los CVE.
Seleccionamos el exploit.
Vemos las opciones de configuración y ejecutamos lo necesario y ejecutamos el exploit.
Hemos también obtenido un usuario y una contraseña para acceder por la interfaz de openfire.
Habremos obtenido una consola, usaremos el comando super importante `id` y `hostname` veremos que somos root.

Como segunda opción desde la web de openfire.
Una vez hemos accedido podemos ver la tabla de usuarios dentro de la web y vamos a hacer fuerza hacia los usuarios para poder acceder via ssh.
Después de hacer fuerza bruta hemos accedido por ssh con sus credenciales y ahora haremos la escalada de privilegios vemos que con `sudo -l` tenemos la posibilidad de ejecucion de `dpkg`.
Ejecutamos el binario sobre el que tenemos permisos
`sudo -u pinguinacio /usr/bin/dpkg -l` el -l nos mostrara un menu de ayuda que si en el escribimo `!/bin/bash` optendremos una bash como el usuario que deseamos

Ahoara con el nuevo usuario ejecutamos de nuevo `sudo -l`
tiene una escala de privilegios a traves d eun operador de bash `-eq` que se encuentra en uin archivo .sh en caso de encontarlo se puede explotar.
Ver este video https://www.youtube.com/watch?v=6MvVNrGcN2M.

