
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
Buscaremos archivos que sean extensión `.aspx` en nuestro equipo por si alguno tiene correspondencia con `cmd` para poder intentar ejecutar una *WebShell*
```shell
locate .aspx | grep cmd
```

