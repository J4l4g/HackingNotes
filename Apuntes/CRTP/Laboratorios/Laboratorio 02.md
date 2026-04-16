## Tareas
1.- Enumerar los siguientes campos para el dominio *dollarcorp*
- [ ] Listar las ACL del grupo de Domain Admins
- [ ] Obtener las ACL donde el usuario tenga permisos interesantes
- [ ] Analizar los permisos para el usuario en *BloodHound*

**Flag Derechos de Active Directory para el grupo RDPUsers en los usuarios llamados ControlUser**


## Enumeración del dominio *dollarcorp*
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

## Enumeración
### Obtener las ACLs del grupo Domain Admins
```shell
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

### Obtener las ACLs interesantes que tenga el usuario
```shell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student97"}
```

No obtenemos información relevante, vamos a listar los grupos a los que pertenece este usuario
```sehll
whoami /groups
```

Nuestro usuario pertenece al grupo de RDPUsers, ahora podemos ver las ACL interesantes que estén asignadas al grupo.
```shell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

## BloodHound
### Analizar los permisos del usuario usando BloodHound
Obtendremos toda la informacion del dominio con 
