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



