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
### Obtener a los grupos que pertenece un usuario
```shell
Get-DomainGroup -UserName "<user>"
```
### Obtener todos los grupos que contengan la palabra *admin*
```shell
Get-DomainGroup *admin*
```
### Obtener todos los miembros pertenecientes al grupo Domain Admins
```shell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```
### Listar todos los grupos locales de una maquina
Para poder ejecutar esta tarea será necesario tener privilegios de Administrador en todas aquellas maquinas que no sean Doamain Controllers
```shell
Get-NetLocalGroup -ComputerName <maquina>
```
### Listar los miembros que pertenecen al grupo Administradores de una maquina
Para poder ejecutar esta tarea será necesario tener privilegios de Administrador en todas aquellas maquinas que no sean Domain Controllers
```shell
Get-NetLocalGroupMember -ComputerName <maquina> -GroupName Administrators
```
### Obtener una lista de usuarios logeados en una maquina
Para realizar esta acción es necesario tener privilegios Administrador en la maquina objetivo
```shell
Get-NetLoggedon -ComputerName <maquina>
```
### Obtener usuarios logeados en un equipo de forma local
Se necesitara un inicio de sesion remoto en el equipo destino
```shell
Get-LoggedonLocal -ComputerName <maquina>
```
### Obtener el ultimo usuario logeado en una maquina
Se necesitara un acceso en la maquina mediante acceso remoto y privilegios de administrador
```shell
Get-LastLoggedonLocal -ComputerName <maquina>
```
### Buscar recursos compartidos en el dominio actual
```shell
Invoke-ShareFinder -Verbose
```
### Buscar ficheros sensibles en los equipos del dominio
```shell
Invoke-FileFinder -Verbose
```
### Obtener todos los servidores de ficheros en el dominio
```shell
Get-NetFileServer
```
