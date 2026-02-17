
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.95.210 -oG allPorts
```

```shell
[*] IP Address: 10.129.95.210
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49681,49685,49700,49866
```

```shell
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49681,49685,49700,49866 -sCV 10.129.95.210 -oN targeted
```

## SMB
Al tener el `SMB` abierto enumeraremos el servicio con [[NETEXEC]]
```shell
nxc smb 10.129.95.210
```

### Listar recursos compartidos

```shell
smbclient -L 10.129.95.210 -N      
```

No vemos ningún tipo de información de recursos compartidos

## DNS
Vamos a ver si es vulnerable al ataque de transferencia de zona

### Ataque de transferencia de zona

