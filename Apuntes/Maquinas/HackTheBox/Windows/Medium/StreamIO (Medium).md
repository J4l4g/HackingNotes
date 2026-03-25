
# Reconocimiento
```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.3.119 -oG allPorts
```

```shell

 [*] IP Address: 10.129.3.119
 [*] Open ports: 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389,49667,49677,49678,49708,49734
```

```shell
nmap -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389,49667,49677,49678,49708,49734 -sCV -oN targeted 10.129.3.119
```

## SMB

```shell
nxc smb 10.129.13.11 
```

### Recursos compartidos

```shell
nxc smb 10.129.13.11 --shares
```

```shell
smbclient -L 10.129.13.11 -N
```

No podemos enumerar ningún recurso compartido

## Kerberos
Haremos una enumeracion de uusuarios a traves de kerberos
```shell
kerbrute userenum --dc 10.129.13.11 -d streamIO.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 
```

Encontrando los usuarios
```ad-info
martin@streamIO.htb
administrator@streamIO.htb
```

