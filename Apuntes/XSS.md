# XSS reflejado en HTML sin codificación
Esta vulnerabilidad se acontece cuando te refleja el input introducido en el HTML
Ejecutaremos por ejemplo un 
```js
<script>alert(0)</script>
```

# XSS almacenado en HTML sin codificación
Esta vulnerabilidad se acontece cuando se guarda el script en la base de datos y una vez te metes en esa sección de nuevo te muestra el código inyectado

# XSS DOM con 'document.write' y 'location.search'
Esta vulnerabilidad se acontece cuando se usa la función `document.write` que tiene una función para introducir datos en la pagina utilizando la llamada a `location.search` que inserta un valor el cual podemos controlar a través de la URL

DOM(Document Object Model) o lo que es lo mismo una estructura de un documento HTML
