
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

Buscaremos si tenemos capacidad de escritura sobre el *FTP* ya que hemos visto que este esta relaccionado a la web que se muestra ya que el contenido a mostrar es identico el uno al otro.

Crearemso un archivo llamado `prueba.txt` con contenido `whoami` y lo probaremos a subir al *FTP*

