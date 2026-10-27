# 1. Pentesting web

## Vulnerabilidades principales

|Técnica|Detección y variantes|Resumen clave|
|---|---|---|
|SQL Injection (SQLi)|Errores SQL, cambios de respuesta y condiciones booleanas. Tipos: error-based, UNION-based, boolean-blind, time-based y stacked queries.|La entrada modifica la consulta SQL. Impacto: lectura o modificación de datos y, en ciertos escenarios, otros efectos sobre el servidor. Mitigación: consultas parametrizadas y mínimo privilegio.|
|XSS|Identificar si la entrada controlada se interpreta como código en el navegador. Tipos: reflejado, almacenado y DOM-based.|Ejecución de JavaScript en el origen de la aplicación. Puede afectar a usuarios, sesiones y acciones que realizan. Mitigación: codificación contextual de salida, sanitización adecuada y CSP como defensa adicional.|
|IDOR / BOLA|Cambiar identificadores de objetos en URL, cuerpo o parámetros y comprobar autorización.|Acceso a objetos de otros usuarios por falta de autorización a nivel de objeto. IDOR es el término habitual; BOLA es frecuente en APIs. Mitigación: comprobar permisos en el servidor en cada operación.|
|Escalada horizontal|Acceso a recursos de otro usuario con privilegios similares.|Fallo de aislamiento entre usuarios. No requiere necesariamente conseguir más permisos.|
|Escalada vertical|Acceso a funciones administrativas o privilegios superiores.|Fallo de autorización por función o rol. Revisar endpoints y operaciones privilegiadas.|
|CSRF|Comprobar si una acción autenticada puede inducirse desde otro sitio sin protección suficiente.|El navegador envía una petición usando la sesión de la víctima. Mitigación: tokens CSRF, cookies `SameSite` y validaciones de origen apropiadas.|
|SSRF|Parámetros que provocan peticiones del servidor a URLs o recursos controlados.|El servidor actúa como intermediario hacia destinos externos o internos. Riesgo de acceso a servicios internos y metadatos cloud. Mitigación: listas permitidas de destinos, controles de salida y validación de redirecciones/DNS.|
|Path Traversal|Entradas que influyen en rutas de archivos; comprobar normalización y restricciones de directorio.|Acceso a archivos fuera de la ruta autorizada. Mitigación: rutas controladas, canonicalización y permisos mínimos.|
|LFI / RFI|Parámetros que incluyen recursos locales o remotos en la aplicación.|LFI: inclusión de archivos locales; RFI: inclusión remota cuando la plataforma y configuración lo permiten. Pueden causar exposición de información o ejecución de código según el contexto.|
|Subida de archivos|Revisar extensión, MIME, firma, nombre, ubicación, permisos y tratamiento posterior.|Subir un fichero no implica que pueda ejecutarse. El riesgo depende de cómo se almacene, procese y sirva.|
|XXE|XML procesado con entidades externas habilitadas; comprobar cómo resuelve referencias el parser.|Puede permitir lectura de archivos o peticiones desde el servidor. Mitigación: deshabilitar entidades externas y usar parsers seguros.|
|SSTI|Entradas interpretadas por un motor de plantillas; errores o comportamiento evaluado en el servidor.|La expresión se evalúa en el motor de plantillas. Puede llegar a exposición de datos o ejecución de código dependiendo del motor y del contexto.|
|Command Injection|Entradas que llegan a comandos del sistema; errores, diferencias de salida o comportamiento temporal.|La aplicación incorpora datos controlados por el usuario en comandos del sistema. Mitigación: evitar shells, usar APIs seguras y validar argumentos.|
|LDAP Injection|Entradas que alteran filtros o consultas LDAP.|Manipulación de consultas al directorio, con posible bypass de filtros o autenticación según la implementación.|
|Open Redirect|Parámetros de retorno o redirección que aceptan destinos externos no autorizados.|Puede facilitar phishing y cadenas de ataque. Mitigación: destinos permitidos y validación estricta.|
|HTTP Request Smuggling|Diferencias de interpretación de los límites de una petición entre proxy y servidor.|Puede desincronizar conexiones y afectar a peticiones posteriores. Revisar especialmente el tratamiento de `Content-Length` y `Transfer-Encoding`.|
|CORS mal configurado|Revisar `Access-Control-Allow-Origin`, credenciales y validación del origen.|Puede permitir que orígenes no autorizados lean respuestas. CORS no sustituye a la autenticación ni a la autorización.|
|Clickjacking|Revisar si la aplicación puede cargarse en un iframe no autorizado.|La víctima puede ser inducida a interactuar con la interfaz. Mitigación: `Content-Security-Policy: frame-ancestors` y controles compatibles.|
|Information Disclosure|Respuestas, errores, cabeceras, archivos, backups, mapas de código o metadatos.|Exposición de información sensible, versiones o estructura interna. La gravedad depende de qué información se revela y cómo se puede utilizar.|
|ID de sesión predecible / Session Fixation|Revisar generación, rotación y aceptación de identificadores de sesión.|La fijación ocurre cuando la víctima termina usando un identificador conocido por el atacante. Mitigación: renovar la sesión tras autenticarse y gestionar correctamente los identificadores.|
|Broken Authentication|Revisar login, recuperación de contraseña, MFA, bloqueo, sesiones y tokens.|Errores en los mecanismos de identidad pueden permitir suplantación o toma de cuentas.|
|Race Condition|Repetir operaciones concurrentes y comprobar si se rompe una condición de negocio.|Dos operaciones simultáneas pueden eludir controles o provocar inconsistencias, como duplicar una acción.|
|Business Logic|Probar límites, secuencias de pasos, descuentos, cantidades, roles y estados de transacción.|La aplicación funciona técnicamente, pero su lógica permite un resultado no autorizado. No siempre hay un payload especial.|

## SQLi: chuleta de variantes

|Variante|Cómo reconocerla|
|---|---|
|Error-based|El error o la respuesta revela información sobre la consulta SQL.|
|UNION-based|Se pueden combinar resultados de otra consulta con la respuesta original.|
|Boolean-blind|La respuesta cambia según una condición verdadera o falsa.|
|Time-based|La respuesta presenta diferencias temporales reproducibles según una condición.|
|Stacked queries|El contexto permite ejecutar varias sentencias; depende del driver y la configuración.|
|Second-order SQLi|La entrada se almacena primero y provoca la inyección cuando se utiliza posteriormente en otra consulta.|

Recuerda: que un parámetro acepte una comilla o produzca un error no demuestra por sí solo una SQLi. Debes confirmar que la entrada altera la lógica de la consulta.

## XSS: chuleta de contexto

|Tipo|Característica|
|---|---|
|Reflected|La entrada aparece en la respuesta inmediata.|
|Stored|La entrada queda almacenada y se ejecuta cuando otro usuario consulta el contenido.|
|DOM-based|La vulnerabilidad se produce en el código del navegador al procesar datos inseguros.|
|Contexto HTML|Importa si la entrada se introduce en texto, atributos, etiquetas o estructura HTML.|
|Contexto JavaScript|Requiere codificación apropiada para el contexto JavaScript concreto.|
|CSP|Defensa en profundidad; no sustituye a corregir la inyección.|

# 2. APIs, autenticación y JWT

|Concepto|Detección / variantes|Resumen clave|
|---|---|---|
|REST API|Revisar métodos HTTP, endpoints, parámetros, respuestas y roles.|Aplicar las pruebas web a cada endpoint; la API debe aplicar autorización en el servidor.|
|GraphQL|Revisar esquema, introspección, resolvers, mutaciones, profundidad y coste de consultas.|Puede exponer datos o funciones si la autorización es insuficiente. La introspección no es por sí sola una vulnerabilidad.|
|BOLA|Manipular IDs de recursos y comparar accesos entre identidades autorizadas.|Falta de autorización a nivel de objeto.|
|BFLA|Acceder a funciones de otros roles o niveles de privilegio.|Fallo de autorización a nivel de función.|
|Mass Assignment|Enviar propiedades adicionales en JSON y comprobar cuáles acepta el servidor.|El cliente puede modificar atributos que no debería controlar, como roles o estados.|
|Excessive Data Exposure|Comparar los campos devueltos con los que necesita el usuario.|La API revela información innecesaria. El frontend no debe ser el único filtro de datos sensibles.|
|Rate Limiting|Probar límites de peticiones de forma controlada y revisar respuestas.|Controles insuficientes pueden facilitar abuso de recursos, enumeración o ataques de fuerza bruta.|
|JWT|Revisar cabecera, payload, firma, algoritmo, claims y validación del servidor.|El token suele estar codificado, no cifrado. Modificar el payload no permite alterar legítimamente los datos si la firma se valida correctamente.|
|JWT `alg`|Comprobar que el servidor restringe los algoritmos permitidos.|La verificación debe usar una configuración segura y no confiar ciegamente en el algoritmo indicado por el cliente.|
|JWT `exp` / `nbf`|Revisar expiración, inicio de validez y tolerancia de reloj.|Controlan cuándo puede aceptarse el token.|
|JWT `iss` / `aud` / `sub`|Validar emisor, destinatario y sujeto.|Evitan aceptar tokens emitidos por fuentes incorrectas o destinados a otro servicio.|
|OAuth 2.0|Revisar scopes, redirect URI, flujo, estado y PKCE según el tipo de cliente.|Es un protocolo de autorización; no equivale por sí solo a autenticación.|
|OpenID Connect|Revisar validación del ID token, `nonce`, emisor y audiencia.|Añade una capa de identidad sobre OAuth 2.0.|
|MFA|Revisar recuperación, enrolamiento, cambios de dispositivo y rutas alternativas.|La protección debe mantenerse también en los flujos secundarios.|
|Brute Force / Credential Stuffing|Comprobar límites, bloqueos, detección de automatización y reutilización de credenciales en pruebas autorizadas.|Fuerza bruta prueba combinaciones; credential stuffing reutiliza credenciales previamente filtradas.|
|IDOR + JWT|Comparar la identidad del token con la autorización sobre el objeto solicitado.|Un JWT válido no demuestra que el usuario tenga permiso sobre todos los objetos.|
|CSRF + cookies|Revisar cookies de sesión, `SameSite`, tokens y comprobaciones de origen.|Una cookie autenticada puede enviarse automáticamente; el riesgo depende del flujo y las defensas aplicadas.|

# 3. Active Directory y Kerberos

## Técnicas de ataque

|Técnica|Requisitos / detección|Resumen clave|
|---|---|---|
|Enumeración de AD|Identificar dominio, usuarios, grupos, equipos, SPN, permisos y relaciones de confianza.|Construye el mapa del dominio antes de elegir una ruta de ataque.|
|Kerberoasting|Solicitar tickets TGS para cuentas con SPN y evaluar la fortaleza de sus secretos mediante pruebas offline autorizadas.|Busca recuperar contraseñas débiles de cuentas de servicio. No requiere necesariamente privilegios de administrador de dominio.|
|AS-REP Roasting|Identificar cuentas con la preautenticación Kerberos deshabilitada.|Permite obtener material susceptible de ataque offline en las cuentas afectadas.|
|Pass-the-Hash (PtH)|Disponer de material NTLM utilizable y de un servicio/protocolo compatible.|Reutiliza material de autenticación sin conocer necesariamente la contraseña en texto claro.|
|Pass-the-Ticket (PtT)|Disponer de un ticket Kerberos utilizable en el contexto adecuado.|Reutiliza tickets de autenticación.|
|Overpass-the-Hash|Disponer de material de clave compatible con el mecanismo Kerberos y el entorno.|Utiliza material derivado de credenciales para obtener autenticación Kerberos; no es idéntico a PtH.|
|Golden Ticket|Compromiso del secreto `krbtgt` del dominio.|Permite forjar TGT; es un escenario de compromiso grave del dominio.|
|Silver Ticket|Secreto de una cuenta de servicio y condiciones compatibles con el servicio objetivo.|Permite forjar tickets de servicio para escenarios concretos; su alcance depende del servicio y la validación.|
|DCSync|Permisos de replicación de directorio suficientes.|Simula operaciones de replicación para obtener material de credenciales. No exige necesariamente ejecutar código en el DC.|
|ACL Abuse|Derechos efectivos sobre objetos AD: `GenericAll`, `GenericWrite`, `WriteDACL`, `WriteOwner`, entre otros.|Los permisos sobre objetos pueden permitir modificar atributos, delegar permisos o controlar identidades.|
|RBCD|Control suficiente para modificar la configuración de delegación basada en recursos y condiciones Kerberos compatibles.|Puede permitir suplantación mediante delegación Kerberos en el recurso afectado.|
|Unconstrained Delegation|Equipos configurados para delegación no restringida y condiciones de autenticación relevantes.|Riesgo de exposición de credenciales Kerberos cuando se producen determinadas autenticaciones.|
|Constrained Delegation|Configuración de delegación a servicios concretos y permisos correspondientes.|Limita los servicios a los que se puede delegar, aunque una configuración insegura puede ser explotable.|
|AD CS|Revisar plantillas, permisos de inscripción, EKU, SAN, aprobación y emisión de certificados.|Configuraciones inseguras pueden facilitar la suplantación de identidades o el escalado de privilegios.|
|Trust Abuse|Revisar confianza entre dominios/bosques, dirección, permisos y filtrado de SID.|Una confianza no implica acceso automático: deben cumplirse las condiciones de autenticación y autorización.|
|BloodHound|Recopilar relaciones, permisos y sesiones con un alcance autorizado.|Modela rutas potenciales de ataque; cada arista y condición debe validarse.|
|Password Spraying|Pruebas limitadas y autorizadas sobre cuentas, respetando bloqueos.|Una contraseña común se prueba en múltiples cuentas; busca reducir el riesgo de bloqueo frente a probar muchas contraseñas en una cuenta.|
|Credential Dumping|Acceso y permisos suficientes sobre sistemas o almacenes de credenciales.|Extracción de secretos almacenados o presentes en memoria. Implica un riesgo alto y requiere controles estrictos.|
|LAPS|Revisar implementación, ACL y permisos de lectura de contraseñas locales.|Gestiona contraseñas de administrador local; los permisos excesivos pueden exponerlas.|
|GPO Abuse|Revisar quién puede modificar o vincular GPO y qué equipos/usuarios las reciben.|Una GPO mal protegida puede extender cambios privilegiados a múltiples sistemas.|
|Privileged Groups|Revisar pertenencias directas e indirectas, permisos y cuentas administrativas.|El riesgo depende de los privilegios efectivos y de cómo se utilizan las cuentas.|

## Kerberos vs. NTLM vs. LDAP

|Protocolo|Para qué sirve|Clave para entrevista|
|---|---|---|
|Kerberos|Autenticación mediante tickets.|KDC, TGT, TGS, SPN, preautenticación y delegación.|
|NTLM|Autenticación mediante desafío-respuesta.|Hashes, relay, restricciones y mecanismos de protección.|
|LDAP|Consultar y modificar el directorio según permisos.|No es un protocolo de autenticación equivalente a Kerberos; puede participar en la consulta y administración de AD.|
|LDAPS|LDAP protegido mediante TLS.|Protege el transporte, pero no corrige permisos incorrectos.|
|SMB|Compartición de archivos y otras operaciones de red.|Shares, permisos, autenticación, firma y exposición de información.|
|DNS|Resolución de nombres y localización de servicios.|Registros, zonas, SRV de AD y posibles errores de configuración.|
|RPC|Comunicación entre procesos y servicios remotos.|Exposición de interfaces, permisos y servicios asociados.|
|WinRM|Administración remota de Windows.|Puertos habituales 5985/5986, autenticación y pertenencia a grupos autorizados.|

Regla de oro para AD: una ruta de BloodHound es una hipótesis estructurada, no una garantía de compromiso. Debes comprobar permisos efectivos, condiciones previas, restricciones y el impacto real.

# 4. Redes, Nmap y servicios

## Nmap y reconocimiento

|Concepto|Qué debes recordar|
|---|---|
|Reconocimiento pasivo|Recopilar información sin interactuar directamente con los objetivos, según el alcance y las fuentes disponibles.|
|Reconocimiento activo|Interactuar con hosts, servicios y aplicaciones para identificar la superficie expuesta.|
|TCP SYN scan (`-sS`)|Escaneo TCP que normalmente no completa el handshake. Requiere privilegios apropiados para el modo de escaneo.|
|TCP connect (`-sT`)|Utiliza la conexión TCP del sistema operativo y completa el establecimiento de conexión.|
|UDP scan (`-sU`)|Comprueba servicios UDP; los silencios y timeouts hacen que la interpretación sea más difícil.|
|Service detection (`-sV`)|Intenta identificar el servicio y su versión; no garantiza que la versión detectada sea exacta.|
|NSE (`-sC`)|Ejecuta el conjunto predeterminado de scripts de Nmap; algunos scripts pueden generar actividad significativa.|
|Open|El servicio parece aceptar conexiones en el puerto.|
|Closed|El host responde, pero no hay servicio escuchando en ese puerto.|
|Filtered|Los filtros impiden determinar con claridad si está abierto o cerrado.|
|Banner grabbing|Recopilar información anunciada por un servicio; el banner puede ser falso o estar desactualizado.|
|Port forwarding|Redirigir tráfico de una conexión o interfaz a otra; el riesgo depende del acceso y las reglas configuradas.|
|Pivoting|Utilizar un sistema accesible para alcanzar otra red autorizada en el alcance.|
|Tunneling|Transportar tráfico mediante otro protocolo o canal. No equivale necesariamente a pivotar.|
|VLAN|Segmentación lógica de redes; su efectividad depende de la configuración de switches, routing y controles de acceso.|
|Firewall|Filtra tráfico según reglas; puede bloquear, permitir o registrar conexiones.|
|NAT|Traducción de direcciones; no constituye por sí misma un control de seguridad suficiente.|

## Puertos y servicios habituales

|Puerto|Servicio|Qué revisar|
|---|---|---|
|21|FTP|Credenciales, FTP anónimo y cifrado.|
|22|SSH|Autenticación, claves, algoritmos y permisos.|
|25|SMTP|Relay, autenticación y configuración.|
|53|DNS|Transferencia de zona, registros y configuración.|
|80 / 443|HTTP / HTTPS|Aplicación web, TLS, cabeceras y controles de acceso.|
|88|Kerberos|Autenticación y configuración del dominio.|
|110 / 995|POP3 / POP3S|Autenticación y protección del transporte.|
|143 / 993|IMAP / IMAPS|Autenticación, TLS y exposición.|
|135|MS RPC|Interfaces RPC y servicios expuestos.|
|139 / 445|NetBIOS / SMB|Shares, permisos, firma y enumeración.|
|161|SNMP|Comunidad, versión, información expuesta y controles de acceso.|
|389 / 636|LDAP / LDAPS|Consultas, autenticación, permisos y cifrado.|
|3306|MySQL|Exposición, permisos y configuración.|
|3389|RDP|Exposición, autenticación y controles de acceso.|
|5432|PostgreSQL|Acceso de red, roles y autenticación.|
|5985 / 5986|WinRM|Administración remota y permisos.|
|8080 / 8443|HTTP alternativo|Consolas, aplicaciones y servicios secundarios.|
|9200|Elasticsearch|Autenticación, exposición y acceso a índices.|
|27017|MongoDB|Autenticación, permisos y exposición de bases de datos.|

## Redes y protocolos

|Concepto|Resumen|
|---|---|
|TCP vs. UDP|TCP proporciona una conexión fiable y ordenada; UDP no garantiza entrega ni orden.|
|DNS|Resuelve nombres y puede revelar infraestructura mediante registros y consultas mal configuradas.|
|ARP|Resuelve direcciones IPv4 a MAC dentro del segmento local; susceptible a suplantación si no existen controles apropiados.|
|DHCP|Asigna configuración de red automáticamente.|
|TLS|Protege la comunicación mediante cifrado, integridad y autenticación según la configuración.|
|Certificado TLS|Vincula una identidad con una clave pública; revisar nombre, cadena de confianza, vigencia y uso.|
|SNI|Permite indicar el nombre del servidor durante la negociación TLS.|
|HTTP/1.1 vs. HTTP/2 vs. HTTP/3|Cambian el transporte y la multiplexación; HTTP/3 usa QUIC sobre UDP.|
|Proxy inverso|Recibe peticiones y las dirige a servidores internos.|
|WAF|Filtra tráfico web según reglas y detecciones; puede producir falsos positivos y negativos.|
|VPN|Crea un canal protegido entre extremos o redes; no elimina los riesgos del equipo o servicio conectado.|
|DNS Zone Transfer|Transferencia de datos de zona entre servidores DNS; si está abierta indebidamente puede exponer registros.|
|MITM|Interposición entre partes de una comunicación; requiere condiciones que permitan interceptar o suplantar el canal.|
|HTTP Request Smuggling|Desacuerdo de parsing entre intermediarios y backend, especialmente con límites de peticiones.|

# 5. Linux: escalada de privilegios y enumeración

|Técnica / elemento|Qué comprobar|
|---|---|
|SUID / SGID|Binarios con permisos especiales que ejecutan con identidad efectiva del propietario o grupo.|
|sudo|Reglas de ejecución privilegiada, usuarios autorizados, argumentos y comandos permitidos.|
|Linux capabilities|Capacidades individuales asignadas a procesos o ejecutables; pueden otorgar privilegios específicos.|
|Cron / timers|Tareas periódicas, propietario, scripts ejecutados y permisos de escritura.|
|Servicios systemd|Usuario del servicio, unidades, rutas, permisos y configuraciones.|
|PATH hijacking|Uso inseguro de rutas de búsqueda de ejecutables por procesos privilegiados.|
|Permisos de archivos|Propietario, grupo, lectura, escritura y ejecución; comprobar el contexto real del proceso.|
|Secretos expuestos|Contraseñas, claves privadas, tokens y ficheros de configuración.|
|Kernel exploit|Verificar versión, parcheado, arquitectura, precondiciones y mitigaciones antes de concluir que es explotable.|
|Contenedores|Privilegios, montajes, capacidades, socket del runtime y aislamiento.|
|Enumeración de procesos|Procesos, argumentos, usuarios, puertos y relaciones entre servicios.|
|Escalada local|Obtener privilegios superiores aprovechando una vulnerabilidad o configuración insegura del host.|

Comandos de reconocimiento que deberías saber interpretar:

```
id
sudo -l
uname -a
ps aux
ss -lntup
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```

No memorices únicamente la salida. Explica qué revela cada comando, qué limitaciones tiene y qué hipótesis investigarías a continuación.

# 6. Windows: enumeración y escalada

|Técnica / elemento|Qué comprobar|
|---|---|
|Privilegios de usuario|Identidad efectiva, grupos y privilegios asignados.|
|Servicios vulnerables|Permisos sobre el servicio, ejecutable, directorios y parámetros.|
|Unquoted Service Paths|Rutas de servicios sin comillas que contienen espacios; solo son vulnerables si se dan condiciones concretas de búsqueda y escritura.|
|Scheduled Tasks|Propietario, permisos, acciones y archivos invocados.|
|AlwaysInstallElevated|Configuración que, si está habilitada de forma insegura en los ámbitos correspondientes, puede permitir instalaciones con privilegios elevados.|
|Stored Credentials|Secretos almacenados en archivos, registros, gestores de credenciales o configuraciones.|
|UAC|Mecanismo de control de elevación; no equivale a una frontera de seguridad universal entre todos los contextos.|
|LSASS|Proceso sensible que puede contener material de autenticación; revisar exposición y protecciones.|
|SAM|Base local de cuentas y hashes de contraseñas.|
|DPAPI|Protege determinados secretos y datos del usuario o sistema mediante claves y contexto de seguridad.|
|PowerShell|Identidad, configuración, módulos, scripts, transcripción y registros de ejecución.|
|Windows Defender / EDR|Políticas, telemetría, detecciones y controles de ejecución.|
|Windows Firewall|Reglas de entrada/salida y perfiles activos.|
|SMB signing|Protege la integridad de mensajes SMB y dificulta ciertos ataques de relay; el efecto depende de versión y configuración.|
|NTLM relay|Reenvío de autenticación a otro servicio cuando las condiciones permiten aceptar esa autenticación sin una protección suficiente.|

# 7. Red Team, OPSEC y MITRE ATT&CK

|Concepto|Resumen clave|
|---|---|
|Pentest|Busca vulnerabilidades y demuestra su impacto dentro de un alcance acordado.|
|Red Team|Evalúa objetivos de seguridad mediante una simulación de adversario, normalmente con objetivos y reglas de enfrentamiento específicos.|
|Purple Team|Colaboración entre ofensiva y defensa para probar y mejorar detecciones y respuestas.|
|MITRE ATT&CK|Base de conocimiento de tácticas y técnicas de adversarios; ayuda a planificar, describir y evaluar comportamientos.|
|Reconnaissance|Obtener información sobre el objetivo antes o durante la operación.|
|Initial Access|Conseguir un primer acceso al entorno.|
|Execution|Ejecutar código o comandos en un sistema.|
|Persistence|Mantener acceso después de la sesión inicial; requiere autorización expresa.|
|Privilege Escalation|Obtener privilegios superiores.|
|Defense Evasion|Intentar evitar o reducir la detección por los controles de seguridad.|
|Credential Access|Acceder a material de autenticación.|
|Discovery|Identificar sistemas, usuarios, servicios y relaciones.|
|Lateral Movement|Moverse desde un sistema comprometido hacia otros.|
|Collection|Reunir información relevante para el objetivo autorizado.|
|Exfiltration|Transferir información fuera del entorno; debe limitarse a lo acordado y necesario para demostrar el impacto.|
|Command and Control (C2)|Canal de comunicación entre el operador y los sistemas bajo control.|
|OPSEC|Gestión de huellas, riesgos, identidades, infraestructura y exposición durante la operación.|
|EDR|Detección y respuesta en endpoints mediante telemetría y análisis de comportamiento, entre otros mecanismos.|
|AMSI|Interfaz de Windows que permite inspeccionar contenido relevante para antimalware en aplicaciones compatibles.|
|ETW|Infraestructura de trazas de Windows utilizada para telemetría y diagnóstico.|
|Evasión vs. bypass|La evasión intenta evitar detecciones; un bypass elude un control concreto. Ninguno debe darse por conseguido sin evidencia.|
|Rules of Engagement (RoE)|Alcance, horarios, técnicas permitidas, límites, contactos y condiciones de parada.|

Punto importante: si te preguntan por evasión de EDR, no centres toda tu respuesta en una técnica de bypass. Explica cómo medirías la detección, qué telemetría revisarías y cómo documentarías las limitaciones del control.

# 8. Cloud: AWS y Azure

|Concepto|Qué revisar|
|---|---|
|IAM|Roles, usuarios, políticas, permisos excesivos y relaciones de confianza.|
|Least Privilege|Cada identidad debe tener únicamente los permisos necesarios.|
|Excessive Permissions|Permisos que permiten operaciones o accesos innecesarios; valorar si pueden encadenarse hasta un impacto mayor.|
|Exposed Storage|Buckets, blobs o contenedores accesibles públicamente o con políticas incorrectas.|
|Metadata Service|Acceso desde workloads a metadatos e identidades de instancia; revisar protecciones y segmentación.|
|Secrets Management|Secretos en código, variables, imágenes, repositorios y sistemas de configuración.|
|Security Groups / NSG|Reglas de entrada y salida de red demasiado permisivas.|
|SSRF en cloud|Puede alcanzar recursos internos o metadatos si la red y los controles lo permiten.|
|Cloud Logging|Registros de actividad, acceso, cambios de configuración y alertas.|
|Azure AD / Microsoft Entra ID|Roles, aplicaciones empresariales, consentimientos, identidades de workload y políticas de acceso.|
|Service Principals|Credenciales, permisos y asignaciones de roles de las identidades de aplicación en Azure.|
|Federation / Trust|Relaciones entre identidades o proveedores; revisar quién puede asumir qué identidad y en qué condiciones.|
|Cloud Lateral Movement|Encadenamiento de permisos, identidades y servicios; no implica necesariamente moverse entre máquinas.|
|Shared Responsibility Model|La responsabilidad se reparte entre proveedor y cliente según el servicio contratado.|

# 9. Scripting y automatización

|Concepto|Resumen clave|
|---|---|
|Python|Automatizar peticiones, parsear respuestas, procesar resultados y generar evidencias estructuradas.|
|Bash|Automatización en Linux, encadenamiento de comandos y tratamiento de ficheros.|
|PowerShell|Administración, enumeración y automatización en Windows y entornos Microsoft.|
|HTTP requests|Gestionar métodos, cabeceras, cookies, autenticación, códigos y timeouts.|
|Parsing|Extraer datos de JSON, XML, HTML, CSV y salidas de herramientas.|
|Error handling|Tratar excepciones, respuestas inesperadas y fallos de conectividad sin perder resultados.|
|Timeouts / Retries|Evitar bloqueos y gestionar fallos transitorios sin sobrecargar el objetivo.|
|Concurrency|Paralelizar con límites para mejorar rendimiento sin generar tráfico excesivo.|
|False Positives|Validar manualmente hallazgos automatizados y contrastarlos con evidencias independientes.|
|Output|Guardar resultados reproducibles con timestamps, objetivos, estados y errores.|
|Secure Coding|Evitar inyecciones, gestionar secretos correctamente y validar entradas.|
|Git|Control de versiones, revisión de cambios y trazabilidad de herramientas propias.|

# 10. Vulnerability management, CVSS e informes

|Concepto|Resumen clave|
|---|---|
|CVSS|Sistema de puntuación de severidad técnica. No representa por sí solo todo el riesgo de negocio.|
|CVE|Identificador público de una vulnerabilidad concreta.|
|CWE|Clasificación de tipos de debilidades de software.|
|CPE|Sistema de identificación de productos y versiones.|
|OWASP WSTG|Guía de pruebas de seguridad para aplicaciones web.|
|OWASP API Security Top 10|Categorías de riesgos frecuentes en APIs.|
|CIS Benchmarks|Recomendaciones de configuración segura para sistemas y plataformas compatibles.|
|MITRE ATT&CK|Clasificación de tácticas y técnicas ofensivas; no es un sistema de puntuación de vulnerabilidades.|
|CVSS vs. riesgo real|Combina severidad técnica con exposición, criticidad, probabilidad de explotación e impacto empresarial.|
|Hallazgo reproducible|Debe incluir precondiciones, pasos, evidencias y resultados observados.|
|Impacto potencial vs. demostrado|No presentes un escenario hipotético como un impacto que has probado.|
|Remediación|Corregir la causa raíz y no únicamente ocultar el síntoma.|
|Retest|Repetir las pruebas relevantes para confirmar la corrección.|
|Falso positivo|El hallazgo no se sostiene después de verificarlo.|
|Falso negativo|La herramienta o prueba no identifica una vulnerabilidad que sí existe.|
|Scope|Activos, aplicaciones, cuentas y acciones autorizadas.|
|ROE|Reglas de ejecución y límites operativos del ejercicio.|

## CVSS: lo imprescindible

- CVSS Base: características intrínsecas de la vulnerabilidad.
    
- Attack Vector (AV): desde dónde se puede explotar.
    
- Attack Complexity (AC): complejidad de las condiciones de explotación.
    
- Privileges Required (PR): privilegios previos necesarios.
    
- User Interaction (UI): interacción de una víctima necesaria.
    
- Scope (S): si el impacto afecta a una autoridad de seguridad distinta.
    
- Confidentiality (C), Integrity (I), Availability (A): impacto en confidencialidad, integridad y disponibilidad.
    
- Temporal / Threat y Environmental: ajustan la valoración según el esquema y versión de CVSS utilizados, incluyendo condiciones de amenaza y entorno.
    

Recuerda: un hallazgo con CVSS alto no siempre es la prioridad número uno del cliente, y uno con CVSS medio puede ser prioritario si abre una ruta crítica.

# 11. Escenarios de entrevista: método de respuesta

Para los cinco escenarios prácticos que hemos visto, utiliza este esquema mental:

|Escenario|Pasos clave|
|---|---|
|SQLi|Hipótesis → prueba controlada → confirmación → impacto mínimo necesario → corrección.|
|Acceso inicial|Alcance → contexto del host → privilegios → relaciones → ruta de escalada validada.|
|Ruta de AD|Comprobar cada relación → validar requisitos → demostrar el eslabón crítico → documentar mitigaciones.|
|JWT y autorización|Identidad del token → objeto solicitado → autorización por objeto → prueba negativa → corrección.|
|EDR|Reglas de enfrentamiento → análisis de telemetría → prueba controlada → resultado observable → mejora defensiva.|

# 12. Preguntas de criterio profesional

|Área|Resumen de la respuesta esperada|
|---|---|
|Pentest con poco tiempo|Priorizar por exposición, criticidad, riesgo y alcance; documentar las limitaciones de cobertura.|
|Vulnerabilidad crítica en producción|Detener acciones de riesgo, preservar evidencias y comunicar por el canal acordado.|
|Discrepancia con otro técnico|Revisar hechos, reproducir la prueba y actualizar la conclusión según la evidencia.|
|Cinco vulnerabilidades medias|Valorar contexto, encadenamiento e impacto empresarial, no solo la puntuación.|
|Informe técnico|Evidencias reproducibles, precondiciones, impacto real, severidad justificada y mitigación concreta.|
|Datos sensibles|Acceder únicamente a lo necesario y autorizado; no extraer datos reales innecesariamente.|
|Hallazgo fuera de alcance|No continuar sin autorización; informar al responsable según el procedimiento acordado.|
|Falta de conocimiento|Reconocer el límite, formular hipótesis y explicar cómo investigarías y validarías la respuesta.|
|Gestión del equipo|Coordinar tareas, revisar resultados, comunicar bloqueos y mantener trazabilidad.|
|Comunicación con el cliente|Explicar el riesgo de forma técnica y ejecutiva, sin exagerar ni minimizar el impacto.|
