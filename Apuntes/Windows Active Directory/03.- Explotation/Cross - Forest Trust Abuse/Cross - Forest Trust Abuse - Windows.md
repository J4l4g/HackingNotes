#AD 

>Esta vulnerabilidad se puede explotar cuando llegas a un dominio, y no puedes escalar mas para arriba, y hay mas dominios con confianzas
# Enumeración de cuentas SPN asociadas entre dominios
Utilizaremos la herramienta [[POWERVIEW]] usando `Get-DomainUser`

```shell
Get-DomainUser -SPN -Domain <dominio.objetivo> | slect samacountname
```

Nos devolverá una cuenta que este asociada con el otro dominio
# Enumeración de la cuenta encontrada
Para enumerar esta cuenta usaremos de nuevo [[POWERVIEW]]

```shell
Get-DomainUser -Domain <dominio.objetivo> -Identity <user> | slect samacountname,memberof
```

Si por ejemplo en la salida 