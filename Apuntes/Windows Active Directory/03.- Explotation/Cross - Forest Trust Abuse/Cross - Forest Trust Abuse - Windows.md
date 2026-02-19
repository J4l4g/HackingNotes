#AD 

>Esta vulnerabilidad se puede explotar cuando llegas a un dominio, y no puedes escalar mas para arriba, y enumeras 
# Enumeración de cuentas SPN asociadas entre dominios
Utilizaremos la herramienta [[POWERVIEW]] usando `Get-DomainUser`

```shell
Get-DomainUser -SPN -Domain <dominio.objetivo> | slect SamAccountName
```

Nos devolverá una cuenta 
# Enumeración de la cuenta encontrada