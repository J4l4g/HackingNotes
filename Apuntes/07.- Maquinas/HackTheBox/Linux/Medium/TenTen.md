
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.61.155 -oG allPorts
```

Encontramos los puertos `22` y `80` abiertos, haremos un escaneo mas exhaustivo de estos puertos
```shell
nmap -p22,80 -sCV -vvv 10.129.61.155 -oN targeted
```

Veremos que tecnologías son usadas en la web en búsqueda de conocer algo mas sobre ella
```shell
Whatweb http://10.129.61.155
```

Nos sale un error de que no se encunetra una ruta hasta la maquina
![[Pasted image 20260924114546.png]]

Para ello tendremos que añadir la IP y el dominio al `/etc/hosts`
Ahora podemos volver a ejecutar [[WHATWEB]]
![[Pasted image 20260924115151.png]]

Obteniendo resultados como que se esta corriendo un *WordPress 4.7*, accederemos a la web para ver de que se trata y que mas contenido podemos obtener.

Al ser un *WordPress* probaremos a acceder al panel de login, como no conocemos donde se encuentra usaremos el script de [[NMAP]] que nos permite hacer una enumeración de directorios
```shell
nmap -p80 --script http-enum 10.129.61.155 -oN webScan
```

Encontrando el directorio de `wp-login.php` así que accederemos a el



