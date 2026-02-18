>Es una técnica de movimiento lateral/escalada de privilegios en entornos de Active Directory, dirigiéndose a cuentas principales de servicio (SPN) estos son indicadores únicos de `Kerberos`, siendo utilizados para asignar una instancia de servicio a una cuenta

# Ejecución del ataque

Este ataque se puede realizar siempre que:
- Se tengan credenciales de un usuario de dominio valido
- Estés desde un Linux unido al dominio como root con el archivo `keytab`
- Estés dentro del dominio con un usuario autenticado
- Se tenga una shell dentro del dominio como usuario
- Estando como system en un host windows unido al dominio
- Se este desde un host Windows no unido al dominio y se use runas/netonly

# Herramientas

Las herramientas necesarias para poder desarrollar este ataque son:
- [[IMPACKET]] con el modulo `GetUserSPNs` desde Linux
- [[POWERVIEX]] o [[RUBEUS]] desde Windows

# Requisitos previos

Para que el ataque se pueda llevar acabo sin interrupciones será necesario tener unas credenciales de usuario de dominio o un hash `NTLM`, una shell en el contexto de usuario o una cuenta como system


# Fases
## Listar cuentas SPN del dominio

Recopilaremos una lista de las `SPN` del dominio para posteriormente poder extraer los tickets usando la herramienta [[IMPACKET]]

```shell
impacket-GetUserSPNs -dc-ip <IP> <nombre.dominio>/user
```

Obtendremos una lista de los `SPN` ahora para obtener los tickets de est