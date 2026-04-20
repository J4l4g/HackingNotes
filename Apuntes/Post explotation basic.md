# Evasion de politicas de ejecucion PowerShell
```shell
powershell -ep bypass
```

# Iniciar PowerView
```shell
. .PowerView.ps1
```

# Enumeración
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
Get-NetUser | select cn
```

```shell
net users
```

### Informacion especifica sobre un usuario
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
C$
D$
ADMIN$
IPC$
PRINT$
SYSVOL
NETLOGON
```
