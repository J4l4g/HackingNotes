

>https://portswigger.net/web-security/sql-injection/cheat-sheet

#### Como detectarlo
- *Mandando una `'`* -> Busca errores o anomalías que devuelva
- *Condiciones booleanas `OR 1=1`, `OR 1=2`* -> Buscar diferencias o respuestas inusuales
- *Cargas útiles que se basen en tiempo* -> Buscar diferencias en el tiempo de respuesta
- *Cargas útiles de OAST* -> Activa interacciones de red


### UNION atack
#### Se debe cumplir:
- *Las columnas individuales deben devolver el mismo numero de columnas* -> Cuantas columnas de devuelven en la columna original??
- *Los tipos de datos deben de ser compatibles entre si* -> Que columnas devueltas en la consulta son un tipo de dato adecuado para la columna dos??

##### Método 1
Una forma de probarlo es haciendo un `' order by 1--` y en la cabecera de la petición interceptada ver donde cambia el numero de caracteres por ejemplo CONTENT-LENGTH

##### Método 2
La segunda forma de detectar el numero de columnas es enviando en la petición valores nulos como por ejemplo `' union select null--`, si el numero de valores nulos no coincide con el numero de columnas de la base de datos se devolverá un error, se usa `NULL` por que los valores entre cada columna deben de ser compatibles con la columna original, ya que `NULL` es convertible a cualquier tipo de dato y es mas posible que se tenga éxito


#### Bases de datos Oracle
En caso de toparnos con una base de datos de oracle existe una tabla integrada en este llamda `dual`, por ello las consultas del estilo `UNION` deberian ser de la siguiente manera `' union select null from dual--`


##### Determinar que columna contiene caracteres
Para determinar que columna conitiene caracteres podemos usar cualquiera de los metodos anteriormete comenr

Una vez determinada la cantidad de columnas devueltas por la consulta original y hayamos determinado cual de ellas puede contener datos en la cadena con `' union select null,'a',null--`
