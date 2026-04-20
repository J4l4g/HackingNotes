
## Tareas
- [ ] Enumerar todos los dominios del bosque *moneycorp.local*
- [ ] Enumerar las confianzas del dominio *dollarcorp.moneycorp.local*
- [ ] Enumerar las confianzas externas del bosque *moneycorp.local*
- [ ] Identificar las relaciones de confianza externas del dominio *dollarcorp*(Se pueden enumerar las relaciones de confianza para un bosque que confía en el?)

**Flag 4 Dirección de confianza para as relaciones de confianza entre *dollarcorp.moneycorp.local* y *eurocorp.local*

## Enumeración
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

### Enumerar todos los dominios del bosque *moneycorp.local*
Con el siguiente comando obtendremos todos los los dominios relativos al bosque
```shell
Get-ForestDomain
```

Pero también lo podemos enumerar de forma mas resumida con
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local | select TargetName,TrustAttributes,TrustDirection
```
### Enumerar las confianzas del dominio *dollarcorp.moneycorp.local*
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local
```

### Enumerar las confianzas externas del bosque *moneycorp.local*
```shell
Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

### Enumerar las relaciones de confianza externas del dominio
Conociendo solo la información relativa al dominio
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local | select TargetName,TrustAttributes,TrustDirection
```

Y conociendo tambien las relacciones externas
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local | ? { $_.TrustAttributes -match "FILTER_SIDS" }
```

Si intentamos enumerar las 