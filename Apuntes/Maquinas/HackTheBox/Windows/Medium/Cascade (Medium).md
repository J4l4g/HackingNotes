
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.9.250 -oG allPorts
```

```shell
[*] IP Address: 10.129.9.250
[*] Open ports: 53,88,135,139,389,445,636,3268,3269,5985,49154,49155,49157,49158,49165
```

```shell
nmap -p 53,88,135,139,389,445,636,3268,3269,5985,49154,49155,49157,49158,49165 -sCV 10.129.9.250 -oN targeted
```

```shell
nxc smb 10.129.9.250
```

```ad-info
(name:CASC-DC1) (domain:cascade.local)
```


## RPC
```shell
rpcclient -U "" 10.129.9.250 -N
```

```shell
enumdomusers
```

```shell
rpcclient -U "" 10.129.9.250 -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users.txt
```

Validación de usuarios con  [[KERBRUTE]]
```shell
kerbrute userenum --dc 10.129.9.250 -d cascade.local users.txt
```

Validando los usuarios se nos quedaría la siguiente lista
```ad-info
arksvc
s.smith
r.thompson
util
j.wakefield
s.hickson
j.goodhand
a.turnbull
e.crowe
d.burman
BackupSvc
j.allen
```

Siempre que tenemoos una lista de usuarios validos tenemos que probar el ataque de [[AS-REP Roasting]]

### AS-REP Roast
```shell
impacket-GetNPUsers cascade.local/ -usersfile users.txt -no-pass
```

Ningún usuario es susceptible a este ataque

### User as Password
```shell
nxc smb 10.129.9.250 -u users.txt -p users.txt --no-bruteforce
```

Ningún usuario utiliza su usuario como contraseña
Probamos ahora haciendo fuerza bruta
```shell
nxc smb 10.129.9.250 -u users.txt -p users.txt
```

No pertenece a ninguno

## LDAP
```shell
ldapsearch -x -H ldap://10.129.9.250 -b "DC=cascade,DC=local"
```

Buscar usuarios
```shell
ldapsearch -x -H ldap://10.129.9.250 -b "DC=cascade,DC=local" | grep -i userprincipalname
```

