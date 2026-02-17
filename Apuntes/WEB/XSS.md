# XSS reflejado en HTML sin codificación
Esta vulnerabilidad se acontece cuando te refleja el input introducido en el HTML
Ejecutaremos por ejemplo un 
```js
<script>alert(0)</script>
```

# XSS almacenado en HTML sin codificación
Esta vulnerabilidad se acontece cuando se guarda el script en la base de datos y una vez te metes en esa sección de nuevo te muestra el código inyectado

# XSS DOM
## Con 'document.write' y 'location.search'
Esta vulnerabilidad se acontece cuando se usa la función `document.write` que tiene una función para introducir datos en la pagina utilizando la llamada a `location.search` que inserta un valor el cual podemos controlar a través de la URL

DOM(Document Object Model) o lo que es lo mismo una estructura de un documento HTML

Cuando por ejemplo introduces un
```js
<script>alert(0)</script>
```

Y no te lo muestra en la alerta lo que tendríamos que ver que estructura es la que nosotros estamos controlando en la pagina, por ejemplo una etiqueta de una imagen, lo que haremos cerrar esa etiqueta y a continuación introducir el XSS
```js
"><script>alert(0)</script>
```

## Con ‘innerHTML’ y ‘location.search’
Esta vulnerabilidad se acontece cuando se usa `innerHTML` pudiendo alterar un elemento o múltiples elementos del DOM sin necesidad de afectar al resto de la pagina

Si por ejemplo cuando introduces
```js
<script>alert(0)</script>
```

O probando
```js
<script>alert("test");</script>
```

No te muestra ningún contenido y ni si quiera te lo interpreta y muestra por pantalla
Podemos intentar cargar una imagen con una carga incorrecta y usando los errores introducir código JavaScript
```shell
<img src=0 onerror=alert(0)>
```


## Con # jQuery y ‘location.search’ en #  ‘href’
Esta vulnerabilidad se acontece en los hipervínculos del `HTML` de la pagina web, ya que a la hora de introducir un parámetro lo incluye en un `href` una vez dentro de estos se puede hacer que en vez de hacernos el redirect se pueden ejecutar sentencias

Podemos probar usando
```js
javascript:alert(0)
```


## Con jQuery y evento ‘hashchange’
Esta vulnerabilidad se acontece cuando se usa la función `hashchange`, por ejemplo en una web cuando se observa que se usa el `#` para llamar a una sección de la web, en el caso de que haga mach te lleva a esa zona de la web, haciendo un autoscroll

El primer paso será identificar la vulnerabilidad XSS utilizando en la URL
```js
<img src =0 onerror=alert(0)>
```

Si tu modificando la URL te muestra el error es que es susceptible a XSS, pero si esa URL se la pasas a la victima en su maquina no se acontecerá nada, por que en su maquina no ha habido modificación, para ellos deberemos usar `iframe`

Un `iframe` es un contexto de navegación anidado el cual permite incrustar una pagina HTML en la pagina actual.

# XSS reflejado en atributo con >< codificados
Esta vulnerabilidad se acontece cuando no lo interpreta tal cual si no como cadena normal de texto

Si por ejemplo cuando introduces
```js
<script>alert(0)</script>
```

O probando
```js
<script>alert("test");</script>
```

No te refleja nada por pantalla, entonces tendríamos que ver como se le llama a ese input en la web, por que si en la web utiliza `"` para el input nos lo estaría escapando así que nuestro payload seria el siguiente
```js
"><script>alert(0)</script>
```

Lo que hacemos es cerrar la etiqueta del input donde se mete el contenido, pero para que funcione lo que podemos hacer es añadir mas argumentos a ese input
```js
"onmouseover="alert(0)"
```

Esto lo que hará es que cuando se pase el cursor por encima nos muestre el alert inyectado en el XSS

# XSS almacenado en ‘href’ con comillas codificadas
Esta vulnerabilidad se acontece cuando no lo interpreta tal cual si no como cadena normal de texto

Si por ejemplo cuando introduces
```js
<script>alert(0)</script>
```

O probando
```js
<script>alert("test");</script>
```

En la zona donde se este aconteciendo deberemos de insertar el XSS de la siguiente forma
```js
javascript:alert(0)
```


# XSS reflejado en string JS con corchetes codificados
Esta vulnerabilidad se acontece cuando no lo interpreta tal cual si no como cadena normal de texto

Lo primero que debemos de ver es la forma en la que se esta aconteciendo la llamada en el input, si en el input vemos que después de de introducir `test'` nos deja una comilla abierta lo cual al probar a introducir un XSS nos la cierra y no se interpreta.

Lo que debemos de hacer primero abrir una comilla, luego separado por `;` meter el XSS y despues en otro `var` añadir otro texto aleatorio cerrando la otra comilla
```js
test'; alert(1); var test='testing
```