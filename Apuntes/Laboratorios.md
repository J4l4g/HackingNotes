# Enumeración

*LO - 01*
## Usuarios del dominio
Enumerar todos los usuarios del dominio quedándonos solo con el nombre de usuario
```shell
Get-DomainUser -Domain dollarcorp.moneycorp.local | select samacountname
```

## Equipos del dominio
Enumerar todos los equipos que se encuentran en el dominio
```shell
Get-DomainComputer -Domain dollarcorp.moneycorp.local | Select-Object -ExpandProperty dnshostname
```

## Administradores del dominio
Enumeraremos los usuarios pertenecientes al grupo Domain Admins
```shell
Get-DomainGroupMember -Domain dollarcorp.moneycorp.local -Identity "Domain Admins" -Recurse
```

## Enterprise Admins del dominio
Enumerar los usuarios pertenecientes al grupo Enterprise Admins del dominio
 ```shell
 Get-DomainGroupMember -Identity "Enterprise Admins" -Domain dollarcorp.moneycorp.local -Recurse
 ```

Al no mostrarnos nada enumeraremos las confianzas viendo que pertenecemos al bosque de *moneycorp.local*
```shell
Get-DomainTrust
```

Una vez descubierto el bosque adaptaremos el comando para buscar en el dominio
```shell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local -Recurse
```

## Enumerar carpetas compartidas
Enumeraremos carpetas compartidas donde los nuestro usuario tenga permisos de de escritura
- Primero deberemos enumerar los equipos de la red y guardarlo en un archivo *.txt*
```shell
Get-DomainComputer | select -ExpandProperty dnshostname | Out-File -FilePath "C:\AD\Tools\servers.txt"
```

- Ahora usaremos la herramienta *PowerHuntShares* importando el modulo de *PowerHuntShares.psm1* y ejecutar *HuntSMBShares*
- Cargamos el modulo
```shell
Import-Module C:\AD\Tools\PowerHuntShares.psm1
```

- Y lo ejecutamos
```shell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt
```

- Obteniendo información sobre los recursos compartidos a los que tenemos acceso
- Ahora podremos pasárnoslo a la maquina local y podremos enumerar con la interfaz grafica de *ShareGraph*
- Pudiendo ver ahora que el recurso compartido llamado *IA* tiene permisos de escritura para todos.

*LO - 02*
## Enumerar las ACL para el grupo de Domain Admins
Enumeraremos las ACL del grupo de Domain Admin, en caso de tener una shell iniciada con comandos recientes si da error hay que abrirse una nueva
```shell
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

## Enumerar las ACL donde tenemos permisos
Enumeraremos las ACL de nuestro usuario para saber si hay alguna interesante aplicada sobre el para poder intentar aprovecharnos de ella en un fututo
```shell
 Find-InterestingDomainAcl -ResolveGUIDs | ?{$_IdentityReferenceName -match "student97"}
```

En caso de no ver nada sobre nuestro ususario podemos revisar a que grupos 