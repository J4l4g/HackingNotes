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

Las herramientas necesarias para poder desarollar este ataque son:
- [[IMPACKET]] con el modulo ``