
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

## Enumeración de usuarios con Kerberos

```shell
kerbrute userenum --dc 10.114.135.170 -d SOUPEDECODE.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```