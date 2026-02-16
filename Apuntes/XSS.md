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

Si por ejemplo cuando introduices
```js
<script>alert(0)</script>
```

