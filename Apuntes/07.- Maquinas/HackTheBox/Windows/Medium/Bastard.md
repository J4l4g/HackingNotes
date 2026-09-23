
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.61.47 -oG allPorts
```

Ahora haremos el reconocimiento mas exhaustivo sobre los puertos encontrados `80`, `135` y `49154` usando [[NMAP]]
```shell
nmap -p80,135,49154 -sCV -vvv 10.129.61.47 -oN targeted
```

Observamos que en el puerto `80` encontramos rutas y directorios que existen, también vemos que hay un *Drupal 7*, accederemos a la web a través del navegador.

Antes identificaremos las tecnologías que usa
![[Pasted image 20260923110212.png]]

Ahora ya accederemos a la web a través del navegador
![[Pasted image 20260923110533.png]]

Intentaremos crearnos una cuenta en la sección de `Create new account`
![[Pasted image 20260923110800.png]]

Y al rellenar los campos obtenemos el error que no ha podido ser enviado el mail
![[Pasted image 20260923111059.png]]

En el escaneo de [[NMAP]] encontramos que en el reconocimiento de ficheros y directorios encontramos que hay un archivo `CHANGELOG.txt` que es un documento donde se registran los últimos cambios realizados en la web.
![[Pasted image 20260923112648.png]]

Al observar la versión vemos que es una versión desactualizada de *Drupal* así que con [[SEARCHSPLOIT]] veremos si esta tiene alguna vulnerabilidad conocida
```shell
searchsploit drupal 7.X
```

Encontrando un exploit que permite realizar un *RCE*
![[Pasted image 20260923113806.png]]

Obtendremos mas información sobre el
```shell
searchsploit -x php/webapps/41564.php
```

Nos traeremos el exploit a nuestra maquina
```shell
searchsploit -m php/webapps/41564.php
```

Deberemos de actualizar en el archivo la URL de la web, el endpoint que por defecto viene en `rest_endpoint` este al navegar en la web no se encuentra así que probamos a usar solo `rest` encontrando así el endpoint
Tendremos que cambiar también el nombre del archivo y la data a tramitar
![[Pasted image 20260923120052.png]]

Ejecutamos el exploit
```shell
php drupalExploit.php
```

Se supone que el archivo ya ha sido subido y ya podemos acceder a el
![[Pasted image 20260923120415.png]]

Al acceder a la web y acceder al recurso no obtenemos respuesta a la hora de recibir un 