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


## Enumeración de usuarios validos
### KERBEROS (88)


