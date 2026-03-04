# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.9.24 -oG allPorts
```


```shell
[*] IP Address: 10.129.9.24
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49165,49166
```

```shell
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49165,49166 -sCV -vvv 10.129.9.24 -oN targeted
```


```shell
nxc smb 10.129.9.24
```

## Enumeración recursos compartidos

```shell
nxc smb 10.129.9.24 -u "" -p "" --shares
```

```shell
smbmap -H 10.129.9.24 -u "" -p ""
```

```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$                            Remote IPC
NETLOGON                        Logon server share
Replication     READ            
SYSVOL                          Logon server share
Users                           
```

Listaremos que contiene este recurso compartido
```shell
smbmap -H 10.129.9.24 -u "" -p "" -r Replication
```

Contiene un fichero llamado `active.htb`, seguimos listando su contenido
```shell
smbmap -H 10.129.9.24 -u "" -p "" -r Replication/active.htb
```




