#AD 

# Enumeración de cuentas SPN asociadas entre ellas
Utilizaremos la herramienta [[POWERVIEW]] usando `Get-DomainUser`

```shell
Get-DomainUser -SPN -Domain
```