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

Encontramos una salida potencial de un directorio llamado */dev*
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
