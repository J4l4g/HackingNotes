## Tareas
1.- Enumerar los siguientes campos para el dominio *DollarCorp*
- [ ] Enumerar todas las *Unidades Organizativas (OU)*
- [ ] Enumerar todos los equipos de la Unidad Organizativa *DevOps*
- [ ] Enumerar las GPO
- [ ] Enumerar la GPO aplicada a la Unidad Organizativa *DevOps*
- [ ] Enumerar las *ACL* para las *GPO* de *Applocker* y *DevOps*

**Flag 3 Nombre de la GPO aplicada a la OU StudentMAchines**

## Enumeracion
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

### Enumerar todas las Unidades Organizativas de un dominio
```shell
Get-DomainOU -Domain dollarcorp.moneycorp.local | select name, ou, distinguishedname
```
