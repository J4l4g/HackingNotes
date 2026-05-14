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