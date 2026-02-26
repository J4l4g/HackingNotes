

# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.230.181 -oG allPorts
```

```shell
[*] IP Address: 10.129.230.181
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49667,49678,49683,49703,49741
```

```shell
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49667,49678,49683,49703,49741 -sCV 10.129.230.181 -oN targeted
```


```shell
nxc smb 10.129.230.181
```

```ad-info
support.htb
```

## Enumeración de recursos compartidos

```shell
smbclient -L 10.129.230.181 -N
```

```shell
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
NETLOGON        Disk      Logon server share 
support-tools   Disk      support staff tools
SYSVOL          Disk      Logon server share 
```


```shell
smbmap -H 10.129.230.181 -u none
```

```shell
Disk                                  Permissions     Comment
----                                  -----------     -------
ADMIN$                                NO ACCESS       Remote Admin
C$                                    NO ACCESS       Default share
IPC$                                  READ ONLY       Remote IPC
NETLOGON                              NO ACCESS       Logon server share 
support-tools                         READ ONLY       support staff tools
SYSVOL                                NO ACCESS       Logon server share 
```

### Acceso al recurso compartido




## Enumeración de Usuarios

```shell
kerbrute userenum --dc 10.129.230.181 -d support.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

Encontramos los usuarios
```ad-info
support
guest
administrator
```

