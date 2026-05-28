# Enumeración
## Usuarios del dominio
Enumerar todos los usuarios del dominio quedándonos solo con el nombre de usuario
```shell
Get-DomainUser -Domain dollarcorp.moneycorp.local | select samacountname
```

## Equipos del dominio
Enumerar todos los equipos que se