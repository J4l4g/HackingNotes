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

Podemos continuar haciendo fuzzing con nmap volviendo a usar el script *http-enum* y con *--script-args* indicar a *http-enum* desde donde queremnos que haga el fuzing
```shell
nmap --script http-enum -p80 --script-args http-enum.basepath='/dev' 10.129.49.90
```
