## Enumeración

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.231.149 -oG allPorts`

```shell
[*] IP Address: 10.129.231.149
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018
```

`nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018 -sCV 10.129.231.149 -oN targeted`

### SMB (139, 445)
Enumeración por smb del dominio
`nxc smb 10.129.231.149`

Descubrimos que el dominio es `cicada.htb` lo añadimos al `/etc/hosts` como `10.129.231.149  cicada.htb dc01 dc01.cicada.htb`

Enumeramos si hay recursos compartidos
`nxc smb 10.129.231.149 --shares`

También podemos enumerar con una Null session
`smbclient -L 10.129.231.149 -N`

O usando netexec
`nxc smb 10.129.231.149 --shares -u 'guest' -p ''`

También podemos usar 


```shell
Disk           Permissions     Comment
----          -----------     -------
ADMIN$         NO ACCESS       Remote Admin
C$             NO ACCESS       Default share
DEV            NO ACCESS
HR             READ ONLY
IPC$           READ ONLY       Remote IPC
NETLOGON       NO ACCESS       Logon server share
SYSVOL         NO ACCESS       Logon server share
```

Vemos que tenemos un recurso compartido llamado HR al que podemos acceder a leer su contenido
`smbmap -H 10.129.231.149 -u 'guest' -p '' -r HR`

## Enumeración de usuarios validos
### KERBEROS (88)
Para enumerar los usuarios validos usaremos,
`kerbrute userenum  --dc 10.129.231.149 -d cicada.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt`

Nos descubre los usuarios `guest` y `administrator`


