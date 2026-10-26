```shell
https://billing-one-pentesting.xylemvue.goaigua.com/billingone/
```
# Medium > Cookie de sesión sin un atributo seguro

**Session Cookie (JSESSIONID) Issued Without the Secure Attribute**

CVSS v3.1: `AV:N/AC:H/PR:N/UI:R/S:U/C:L/I:L/A:N` → 4.2 — Medium / Medio

_(AC:H refleja que el HSTS dificulta la explotación. Si en tu informe lo consideras una mitigación efectiva, puedes bajarlo a `C:L/I:N` → 3.1 Low.)_

**Threat**

EN: An attacker positioned on the network path between the client and the application (e.g., rogue Wi-Fi access point, ARP spoofing, or a compromised network device) could trigger or wait for a plain HTTP request to the application’s domain and capture the session cookie in clear text. With the stolen `JSESSIONID`, the attacker could hijack the victim’s authenticated session and impersonate them, accessing or modifying their data without knowing their credentials.

ES: Un atacante situado en la ruta de red entre el cliente y la aplicación (p. ej., un punto de acceso Wi-Fi falso, ARP spoofing o un dispositivo de red comprometido) podría provocar o esperar una petición HTTP en claro hacia el dominio de la aplicación y capturar la cookie de sesión sin cifrar. Con el `JSESSIONID` robado, el atacante podría secuestrar la sesión autenticada de la víctima y suplantarla, accediendo o modificando sus datos sin conocer sus credenciales.

**Description**

EN: The application (Payara Server 6.2025.11) issues the `JSESSIONID` session cookie with the `Path=/billingone; HttpOnly` attributes but without the `Secure` flag, as observed in the `Set-Cookie` header of the HTTP response. Without this flag, browsers may transmit the cookie over unencrypted HTTP connections. The risk is partially mitigated by the `Strict-Transport-Security` header (`max-age=63072000; includeSubDomains; preload`), which does not protect first visits, non-browser clients, or domains not actually included in the browser preload list.

ES: La aplicación (Payara Server 6.2025.11) emite la cookie de sesión `JSESSIONID` con los atributos `Path=/billingone; HttpOnly` pero sin el flag `Secure`, como se observa en la cabecera `Set-Cookie` de la respuesta HTTP. Sin este flag, los navegadores pueden transmitir la cookie a través de conexiones HTTP sin cifrar. El riesgo se mitiga parcialmente con la cabecera `Strict-Transport-Security` (`max-age=63072000; includeSubDomains; preload`), que no protege las primeras visitas, los clientes que no son navegadores ni los dominios que no estén realmente incluidos en la lista de precarga de los navegadores.

![[Pasted image 20261021095954.png]]

**Recommendations**

EN: Set the `Secure` attribute on the session cookie by adding `<secure>true</secure>` inside `<session-config><cookie-config>` in `web.xml`; this also works behind a TLS-terminating proxy. Additionally, configure the `SameSite` attribute (Lax or Strict), disable URL-based session tracking (`<tracking-mode>COOKIE</tracking-mode>`), and verify the domain is submitted to the HSTS preload list, with HTTP access redirected to HTTPS or disabled entirely.

ES: Establecer el atributo `Secure` en la cookie de sesión añadiendo `<secure>true</secure>` dentro de `<session-config><cookie-config>` en el `web.xml`; esto también funciona detrás de un proxy que termine el TLS. Adicionalmente, configurar el atributo `SameSite` (Lax o Strict), deshabilitar el seguimiento de sesión por URL (`<tracking-mode>COOKIE</tracking-mode>`) y verificar que el dominio esté enviado a la lista de precarga HSTS, redirigiendo el acceso HTTP a HTTPS o deshabilitándolo por completo.




# Low > Information Disclosure 
**Information Disclosure: Payara Server Version Exposure via HTTP Response Headers (Running a Version with Known Public Vulnerabilities)**

CVSS v3.1: `AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:N` → 5.3 — Medium / Medio

_(Vector calculado solo sobre la divulgación de versión. No incluye explotación confirmada de los CVEs asociados, ya que no se ha verificado si la consola de administración de Payara es accesible.)_

**Threat**

EN: An attacker could read the `Server` and `X-Powered-By` HTTP response headers to precisely fingerprint the application server as Payara Server 6.2025.11, without needing to authenticate or send any special request. This version is affected by publicly disclosed vulnerabilities, including an admin account takeover via a crafted URL payload and a Server-Side Request Forgery (SSRF) that can leak the administrator’s REST session token. If the Payara Admin Console or REST management interface is reachable, the attacker could use this version information to directly target those vulnerabilities, potentially resulting in full administrative takeover of the application server.

ES: Un atacante podría leer las cabeceras de respuesta HTTP `Server` y `X-Powered-By` para identificar con precisión el servidor de aplicaciones como Payara Server 6.2025.11, sin necesidad de autenticarse ni enviar ninguna petición especial. Esta versión está afectada por vulnerabilidades públicas conocidas, incluyendo una toma de control de la cuenta de administrador mediante un payload en la URL y un Server-Side Request Forgery (SSRF) que permite filtrar el token de sesión REST del administrador. Si la consola de administración de Payara o la interfaz REST de gestión son accesibles, el atacante podría usar esta información de versión para dirigir sus ataques directamente a dichas vulnerabilidades, lo que podría derivar en una toma de control administrativa completa del servidor de aplicaciones.

**Description**

EN: HTTP responses from the application consistently disclose `Server: Payara Server 6.2025.11` and `X-Powered-By: Servlet/6.0 JSP/3.1 (Payara Server 6.2025.11 Java/Azul Systems, Inc./17)`. This exact version is affected by at least two publicly disclosed vulnerabilities: **CVE-2025-14340**, a Cross-Site Scripting (CWE-79) flaw in the REST Management Interface affecting versions up to 6.2025.11 that can mislead an administrator into changing the admin password via a crafted URL; and **CVE-2026-12986**, a critical SSRF in the Admin GUI’s DownloadServlet (versions up to 6.2025.11) that allows exfiltration of the administrator’s `gfresttoken` to an attacker-controlled host, potentially enabling unauthenticated takeover of the Payara admin domain. Exploitability of these CVEs was not confirmed during this engagement, as access to the Admin Console / REST management interface (typically on port 4848) was not verified as reachable from the tested scope.

ES: Las respuestas HTTP de la aplicación revelan de forma consistente `Server: Payara Server 6.2025.11` y `X-Powered-By: Servlet/6.0 JSP/3.1 (Payara Server 6.2025.11 Java/Azul Systems, Inc./17)`. Esta versión exacta está afectada por al menos dos vulnerabilidades públicas conocidas: **CVE-2025-14340**, un fallo de Cross-Site Scripting (CWE-79) en la interfaz REST de gestión que afecta a versiones hasta la 6.2025.11 y permite inducir al administrador a cambiar su contraseña mediante una URL manipulada; y **CVE-2026-12986**, un SSRF crítico en el DownloadServlet de la Admin GUI (versiones hasta la 6.2025.11) que permite exfiltrar el token `gfresttoken` del administrador hacia un host controlado por el atacante, pudiendo derivar en una toma de control no autenticada del dominio de administración de Payara. La explotabilidad de estos CVEs no se confirmó durante esta auditoría, ya que no se verificó si la consola de administración / interfaz REST de gestión (habitualmente en el puerto 4848) es accesible desde el alcance analizado.

![[Pasted image 20261021102158.png]]

**Recommendations**

EN: Suppress or customize the `Server` and `X-Powered-By` headers at the server or reverse-proxy level to avoid revealing exact software and version information. Upgrade Payara Server to the latest patched release in the 6.x or 7.x branch per the official Security Fix List, and ensure the Admin Console and REST management interface are not exposed to untrusted networks, restricting access to trusted administrative segments only (e.g., VPN or internal network).

ES: Suprimir o personalizar las cabeceras `Server` y `X-Powered-By` a nivel de servidor o proxy inverso para evitar revelar información exacta de software y versión. Actualizar Payara Server a la última versión parcheada dentro de la rama 6.x o 7.x según el Security Fix List oficial, y asegurar que la consola de administración y la interfaz REST de gestión no estén expuestas a redes no confiables, restringiendo el acceso únicamente a segmentos administrativos de confianza (p. ej., VPN o red interna).



# Low > Information Disclosure
**Unauthenticated Exposure of Billing API Specification (OpenAPI) Revealing Internal Infrastructure and Sensitive Endpoints**

CVSS v3.1: `AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` → 5.3 — Medium / Medio

**Threat**

EN: An attacker could access the `/openapi` endpoint without authentication and obtain the complete API specification, including internal hostnames, ports, business-critical endpoints (e.g., bill download, account balance retrieval) and their exact parameter names and types. This significantly reduces the reconnaissance effort required to design targeted attacks, such as parameter tampering, IDOR testing, or fuzzing against the billing functionality, and reveals internal infrastructure details that should not be exposed externally.

ES: Un atacante podría acceder al endpoint `/openapi` sin autenticarse y obtener la especificación completa de la API, incluyendo hostnames internos, puertos, endpoints críticos del negocio (p. ej., descarga de facturas, consulta de balance de cuenta) y sus parámetros exactos con nombre y tipo. Esto reduce considerablemente el esfuerzo de reconocimiento necesario para diseñar ataques dirigidos, como manipulación de parámetros, pruebas de IDOR o fuzzing contra la funcionalidad de facturación, y revela detalles de infraestructura interna que no deberían exponerse externamente.

**Description**

EN: The `/openapi` endpoint on `billing-one-pentesting.xylemvue.goaigua.com` returns, without authentication, the full OpenAPI 3.0.0 specification (MicroProfile OpenAPI) for the `billingone` application. The disclosed content includes internal server URLs with what appears to be a Docker container ID (`fc76d28dcd19`), both HTTP (8080) and HTTPS (8181) listener ports, and multiple business endpoints under `/Bill/` (`DownloadBill`, `GetBalance`, `GetBalanceList`) along with their query parameters (`bill_id`, `account`, `from_date`) and response schemas.

ES: El endpoint `/openapi` en `billing-one-pentesting.xylemvue.goaigua.com` devuelve, sin autenticación, la especificación completa OpenAPI 3.0.0 (MicroProfile OpenAPI) de la aplicación `billingone`. El contenido expuesto incluye URLs de servidores internos con lo que parece ser un ID de contenedor Docker (`fc76d28dcd19`), los puertos de escucha HTTP (8080) y HTTPS (8181), y múltiples endpoints de negocio bajo `/Bill/` (`DownloadBill`, `GetBalance`, `GetBalanceList`) junto con sus parámetros de consulta (`bill_id`, `account`, `from_date`) y esquemas de respuesta.
![[Pasted image 20261021121149.png]]

**Recommendations**

EN: Disable the `/openapi` endpoint in production environments, or restrict access to it through authentication and network-level controls (e.g., internal networks or VPN only). If API documentation must remain available, remove the `servers` section or replace internal hostnames and ports with the public-facing endpoint only, avoiding disclosure of internal infrastructure details.

ES: Deshabilitar el endpoint `/openapi` en entornos de producción, o restringir su acceso mediante autenticación y controles a nivel de red (p. ej., solo redes internas o VPN). Si la documentación de la API debe permanecer disponible, eliminar la sección `servers` o sustituir los hostnames y puertos internos únicamente por el endpoint público, evitando revelar detalles de la infraestructura interna.

