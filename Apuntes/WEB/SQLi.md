
#### Como detectarlo
- *Mandando una `'`* -> Busca errores o anomalías que devuelva
- *Condiciones booleanas `OR 1=1`, `OR 1=2`* -> Buscar diferencias o respuestas inusuales
- *Cargas útiles que se basen en tiempo* -> Buscar diferencias en el tiempo de respuesta
- *Cargas útiles de OAST* -> Activa interacciones de red


### UNION atack
#### Se debe cumplir:
- *Las columnas individuales deben devolver el mismo numero de columnas* -> Cuantas columnas de devuelven en la columna original??
- *Los tipos de datos deben de ser compatibles entre si* -> Que columnas devueltas en la consulta son un tipo de dato adecuado para la columna dos??

##### Metodo 1
Una forma de probarlo es haciendo un `order by 1--` y en la cabecera de la petición interceptada ver donde cambia el numero de caracteres por ejemplo CONTENT-LENGTH

##### Metodo 2
La segunda forma de detectar el numero de columnas es enviando en la petición valores nulos como por ejemplo `union select null--`, 