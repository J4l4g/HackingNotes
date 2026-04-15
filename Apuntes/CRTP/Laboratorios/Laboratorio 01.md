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

Enumeración de equipos del dominio
```shell
Get-DomainComputer -Domain dollarcorp.moneycorp.local | Select-Object -ExpandProperty dnshostname
```

##### Enumeracion de administradores del dominio
```shell
Get-DomainGroupMember -Domain dollarcorp.moneycorp.local -Identity "Domain Admins" -Recurse
```

##### Enumeración de administradores empresariales del dominio
```shell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain dollarcorp.moneycorp.local
```

Con el dominio completo no esta pero si comprobaremos si en el bosque los hay, mirando las relaciones de confianza entre dominios, 
```shell
Get-DomainTrust
```

Encontrando como resultado al confianza con `moneycorp.local`, pudiendo modificar el comando para ver los administradores empresariales
```shell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```


## Encontrar una carpeta compartida donde tengamos permisos de escritura
Enumeraremos todos los equipos del dominio actual que muestren el nombre del host DNS y los guardamos en un archivo
```shell
Get-DomainComputer | select -ExpandProperty dnshostname | Out-File -FilePath "C:\AD\Tools\servers.txt"
```

Importaremos el modulo *PowerHuntShares*
```shell
Import-Module C:\AD\Tools\PowerHuntShares.psm1
```

Y ejecutaremos la herramienta
```shell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools\ -HostList C:\AD\Tools\servers.txt
```

