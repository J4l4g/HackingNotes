# Tareas
01.- Enumerar los siguientes campos para el dominio de *dollarcorp*
- [ ] Users
- [ ] Computers
- [ ] Domain Admins
- [ ] Enterprise Administrators

02.- Usando BloodHound identificar la ruta mas corta para llegar a Domain Admin en *dollarcorp*
03.- Encontrar un recurso compartido donde *studen97* tenga permisos de escritura

**Flag SID del miembro del grupo de Enterprise Administrators**

## Enumeración del dominio *dollarcorp* 
Para esta enumeración usaremos *InviShell* y *PowerView*

Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
.\PowerView.ps1 
```

##### Enumeración de usuarios del dominio
```shell
Get-DomainUser -Domain dollarcorp.moneycorp.local | select samaccountname
```