
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

