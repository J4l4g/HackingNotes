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
### Obtener políticas del Dominio acctual
```shell
Get-DomainPolicyData
```
### Obtener politicas de otro Dominio