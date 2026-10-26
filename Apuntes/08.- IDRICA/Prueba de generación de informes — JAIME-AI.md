## 1. Information Disclosure - Stack trace y rutas internas

Revisando la aplicación web de pruebas `https://app.ejemplo.test`, he visto que si se manda una petición con un parámetro con formato incorrecto devuelve un error bastante detallado. Aparece un stack trace de .NET, nombres de clases y una ruta interna del servidor.

No he conseguido acceso al sistema ni he visto credenciales. Es exposición de información técnica que puede ayudar a preparar ataques posteriores. La respuesta se reproduce sin iniciar sesión.

Petición de Burp:

```
GET /api/orders?id=abc HTTP/1.1
Host: app.ejemplo.test
Accept: application/json
```

Respuesta:

```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "Unhandled exception",
  "type": "System.FormatException",
  "source": "OrdersController.cs",
  "path": "D:\\\\Apps\\\\Portal\\\\Controllers\\\\OrdersController.cs"
}
```

Evidencia:  
![[Pasted image 20261026134540.png]]

Impacto que considero: un atacante externo podría conocer detalles de la tecnología y estructura interna, facilitando la preparación de ataques más dirigidos. No se ha demostrado un impacto adicional.

---

## 2. SQL Injection - Parámetro de búsqueda de productos

En el buscador de productos he probado el parámetro `category`. Con el valor normal `Gifts` devuelve productos de esa categoría. Si añado una condición SQL que siempre es verdadera, la aplicación devuelve también productos de otras categorías.

He repetido la prueba y el resultado es consistente. No he extraído datos personales, no he probado escritura ni borrado y no he intentado acceder al sistema operativo.

Petición original:

```
GET /products?category=Gifts HTTP/1.1
Host: shop.ejemplo.test
```

Petición modificada:

```
GET /products?category=Gifts%27%20OR%201%3D1-- HTTP/1.1
Host: shop.ejemplo.test
```

Resultado observado: la respuesta contiene productos que no pertenecen a `Gifts`. La comparación se ha realizado con la respuesta de la petición original.

Evidencia:  
![[Pasted image 20261026134553.png]]


Impacto posible: manipulación de las consultas de base de datos y acceso no autorizado a información, dependiendo de los permisos de la cuenta de base de datos y del contexto de la consulta. La prueba actual demuestra alteración de la lógica de filtrado, pero no demuestra extracción de información sensible.

---

## 3. Missing Security Headers - Cabeceras HTTP de seguridad

Durante la revisión de las respuestas HTTP de la página principal he observado que no se incluyen las cabeceras `Content-Security-Policy` ni `X-Content-Type-Options`.

La página se puede consultar sin autenticación. No he identificado ni demostrado una vulnerabilidad XSS, ejecución de JavaScript inyectado o carga de contenido malicioso. Solo he comprobado la ausencia de las cabeceras en la respuesta revisada; falta confirmar si ocurre también en el resto de páginas.

Respuesta:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Evidencia:  
![[Pasted image 20261026134605.png]]

Impacto posible: la ausencia de estas cabeceras reduce ciertas defensas adicionales del navegador. El impacto real depende del contenido de la aplicación y de la existencia de otras vulnerabilidades; no debe asumirse que esta ausencia permita por sí sola ejecutar código.

---

## 4. IDOR - Acceso a la factura de otro usuario

En el entorno de pruebas he iniciado sesión con el usuario de prueba A y he consultado la factura `1042`. Después he cambiado el identificador por `1043`, que corresponde al usuario de prueba B, manteniendo la misma sesión.

La aplicación ha respondido con HTTP 200 y ha devuelto los datos de la factura del usuario B. He verificado que la factura pertenece al usuario B usando los datos de prueba proporcionados para la auditoría. No he modificado ni eliminado información.

Petición con la sesión del usuario A:

```
GET /api/invoices/1043 HTTP/1.1
Host: billing.ejemplo.test
Cookie: session=[REDACTED]
Accept: application/json
```

Respuesta observada:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "invoiceId": 1043,
  "customer": "Usuario de prueba B",
  "description": "Servicios contratados",
  "amount": 1250.00,
  "status": "Unpaid"
}
```

Evidencia:  
![[Pasted image 20261026134619.png]]

Impacto: un usuario autenticado puede consultar información de facturación de otro usuario modificando el identificador de la petición. La prueba demuestra acceso cruzado entre las dos cuentas de prueba. No se ha probado el acceso a facturas de otros clientes reales ni la modificación de facturas.

---

## Notas para el informe

- Los cuatro hallazgos deben aparecer como vulnerabilidades independientes.
    
- Redactar Threat, Description y Recommendations en español e inglés.
    
- Mantener las cuatro referencias de imagen, cada una en la descripción española de su hallazgo.
    
- No inventar información ni afirmar que se han demostrado impactos que no aparecen en las pruebas.
    
- Para cada hallazgo, calcular CVSS v3.1 solo si se pueden justificar todas las métricas base. Si falta información, indicarlo y explicar qué hay que confirmar.
    
- No reutilizar puntuaciones o vectores de ejemplo.
    
- Mantener recomendaciones concretas y proporcionales a cada vulnerabilidad.