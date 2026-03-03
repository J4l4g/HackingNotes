>Es una técnica de movimiento lateral/escalada de privilegios en entornos de Active Directory que explota una configuración débil en las cuentas de usuario. A diferencia de [[Kerberoasting]], este ataque no se dirige a cuentas con SPN si no a cuentas de usuario que tengan habilitada la opción "Do not require Kerberos preauthentication" .

# Ejecución del ataque

Este ataque se puede realizar siempre que:
- Un usuario válido en el dominio (sin necesidad de privilegios especiales).
- Conectividad con el Controlador de Dominio.
- Capacidad para ejecutar herramientas que puedan solicitar tickets Kerberos.
# Herramientas

Las herramientas necesarias para poder desarrollar este ataque son:
- [[IMPACKET]] con  `GetNPUsers`, desde Linux
- [[RUBEUS]] (con el argumento `/asreproast`) o [[POWERVIEW]] (con el comando `Get-DomainUser -PreauthNotRequired`), desde Windows

# Requisitos previos

Para que el ataque se pueda llevar a cabo sin interrupciones, será necesario tener unas credenciales de un usuario de dominio válido (o una shell en el contexto de ese usuario). No se requiere un hash `NTLM` específico para iniciar el ataque, ya que cualquier usuario autenticado puede consultar el directorio activo.

# Fases
## Enumerar usuarios vulnerables

Primero, debemos identificar qué cuentas de usuario en el dominio tienen la configuración "No requerir preautenticación de Kerberos" habilitada. Con [[IMPACKET]] desde Linux, lo podemos hacer listando directamente y solicitando el ticket para aquellos que no requieren preautenticación.

```shell
impacket-GetNPUsers -dc-ip <IP_DEL_DC> <NOMBRE_DOMINIO>/<USUARIO_VALIDO> -no-pass
```

Este comando intentará obtener el TGT para _todos_ los usuarios. Si el usuario no es vulnerable, la herramienta lo ignorará silenciosamente o dará un error. Si es vulnerable, nos devolverá el ticket cifrado.

## Extraer el ticket (hash) de la cuenta vulnerable

Si queremos apuntar a un usuario específico del que sospechamos que es vulnerable, o para ser más silenciosos, podemos usar:

```shell
impacket-GetNPUsers -dc-ip <IP_DEL_DC> <NOMBRE_DOMINIO>/<USUARIO_VALIDO> -request-user <USUARIO_OBJETIVO> -no-pass
```

Para guardar la salida en un archivo y poder trabajar con ella más tarde, usamos la redirección o la opción `-outputfile`:

