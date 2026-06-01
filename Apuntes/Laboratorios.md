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

## Enumerar las ACL donde tenemos permisos interesante
Enumeraremos las ACL de nuestro usuario para saber si hay alguna interesante aplicada sobre el para poder intentar aprovecharnos de ella en un fututo
```shell
 Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student97"}
```

En caso de no ver nada sobre nuestro usuario podemos revisar a que grupos pertenecemos y hacer la enumeración de las ACL de estos
```shell
whomai /groups
```

En cado de encontrar algún grupo interesante como puede ser el de RDPUsers volvemos a lanzar la enumeración
```shell
 Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

*LO - 03*
## Enumerar las OU del dominio dollarcor.moneycorp.local
Enumeraremos todas las Unidades Organizativas del dominio actual
```shell
Get-DomainOU -Domain dollarcorp.moneycorp.local | select name, ou, distinguishedname
```

Una ves enumeradas todas las Unidades Organizativas lo siguiente que podemos hacer es enumerar todos los equipos que pertenecen a una unidad organizativa
```shell
(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

## Enumerar las GPO
Enumeraremos las GPO implementadas
```shell
Get-DomainGPO | select displayname
```

También podemos enumerar las GPO que están aplicadas sobre una Unidad Organizativa como puede ser DevOps
Para ello lo primero que necesitaremos sera el nombre de la directiva del atributo gplink de la Unidad Organizativa
```shell
(Get-DomainOU -Identity DevOps).gplink
```

Deberemos de copiar el valor que se encuentra entre corchetes incluyendo estos, a continuación ya podremos enumerar las GPO de la Unidad Organizativa
```shell
Get-DoaminGPO -Idenetity '{0BF8D01C-1F62-4BDC-958C-57140B67D147}'
```

*LO - 04*

## Enumerar todos los dominios en bosque *moneycorp.local*
Enumeraremos todos los dominios que se encuentran en el bosque
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local | select TargetName,TrustAttributes,TrustDirection
```

## Enumerar las confianzas del dominio actual
Enumeraremos las confianzas de nuestro dominio pudiendo recoger las confianzas y la dirección relativa de estos
```shell
Get-DomainTrust -Domain dollarcorp.moneycorp.local
```

## Enumerar las confianzas externas al bosque *moneycorp.local*
Enumeraremos todas la confianzas externas del bosque filtrando únicamente por el SID principal
```shell
Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

## Enumerar las relaciones de confianzas externas al dominio *dollarcorp.moneycorp.local*
Habiendo enumerado ya las confianzas del bosque y  y del dominio hemos descubierto que hay un bosque externo vamos a enumerarlo verificar las relaciones de confianza
```shell
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

Dándonos cuenta que no podemos enumerar un bosque o dominio externo al nuestro si intentamos obtener mas información del dominio usando
```shell
Get-DomainForest -Forest eurocorp.local
```

Nos daremos cuenta que no lo podemos enumerar completo ya que no tenemos visibilidad con el

*LO - 05*
