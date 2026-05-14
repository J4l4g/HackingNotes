## Enumeración del Dominio
Lo primero que se debe de hacer al obtener una nueva shell será ejecutar *InviShell* y *PowerView*
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat
```

```shell
. .\PowerView.ps1
```
### Información del Dominio actual
```shell
Get-Domain
```
### Información de otro Dominio
```shell
Get-Domain -Domain <dominio.local>
```
### Obtener SID del Dominio actual
```shell
Get-DoimainSID
```
### Obtener políticas del Dominio actual
```shell
Get-DomainPolicyData
```
### Obtener políticas de otro Dominio
```shell
(Get-DomainPolicyData -domain moneycorp.local).systemaccess
```
### Domain Controllers del Dominio actual
```shell
Get-DomainController
```
### Domain Controller de otro Dominio
```shell
Get-DomainController -Domain moneycorp.local
```
### Obtener una lista de usuarios del Dominio actual
```shell
Get-DomainUser
```
Si queremos filtrar por un usuario en concreto usaremos `-Identity` y el nombre de usuario
Si queremos solo los nombres de usuario podemos usar `| select name`
### Obtener todas las propiedades de un usuario
```shell
Get-DomainUser -Identity student97 -Properties *
```
### Buscar cuentas que sean interesantes por descripcion
```shell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```
### Obtener los equipos del Dominio actual
```shell
Get-DomainComputer | select name
```

Si queremos listar por los equipos que usen un *Server 2022*
```shell
Get-DomainComputer -OperatingSystem "*Server 2022*"
```

Si queremos ver que equipos responder por red mediante ping
```shell
Get-DomainComputer -Ping
```
### Obtener todos los grupos en el Dominio actual
```shell
Get-DomainGroup | select name
```
Si queremos hacerlo sobre otro dominio podemos usar la opción `-Domain`
### Obtener todos los grupos que contengan la palabra *admin*
```shell
Get-DomainGroup *admin*
```
### Obtener todos los miembros pertenecientes al grupo Domain Admins
```shell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```