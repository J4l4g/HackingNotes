# Desde Linux
Podemos usar herramientas como [[NETEXEC]], que nos ayuda  principalmente para la enumeración, validación de credenciales y movimiento lateral en entornos `Windows` y `Active Directory`

Podemos usar además usar la herramienta [[SMBMAP]] que nos permite enumerar y evaluar los recursos `SMB` en sistemas `Windows` y `Active Directory`

También se puede usar [[RPCCLIENT]] que nos permite interactuar con los servicios `RPC` de sistemas `Windows`

La herramienta [[WINDAPSEARCH]] también nos permite realizar una enumeración `Active Directory` a través de `LDAP`

# Desde Windows
Para hacer la enumeración desde Windows lo primero que deberemos de hacer ver los modulos que tenemos importados desde la `Power Shell`, usando el comando `GET-Module` tiene que importado el modulo de `ActiveDirectory`, si no aparece lo importaremos con `Import-Module ActiveDirectory`, volveremos a ejecutar `GET-Module` para ver si se ha cargado correctamente.

