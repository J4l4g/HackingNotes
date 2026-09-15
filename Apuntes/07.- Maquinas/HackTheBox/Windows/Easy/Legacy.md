```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.227.181 -oG allPorts
```

```shell
nmap -p135,139,445 -sCV -vvv 10.129.227.181 -oN targeted 
```

Encontramos los puertos `135`, `139` y `445` abiertos 
Empezaremos listando los recursos compartidos del *SMB* 
```shell
nxc smb 10.129.227.181
```

![[Pasted image 20260915124606.png]]
Vemos que la firma del *SMB* no esta activada al igual que esta *SMBv1* activada, ahora nos iremos con los recursos compartidos
```shell
nxc smb 10.129.227.181 --shares
```

Sin obtener la información ninguna sobre recursos compartidos.
Al ser un *Windows XP* podemos ver la opcion de que se pueda explotar *EternalBlue*, buscaremos un script de [[NMAP]] que nos ayude a identificar si se puede explotar esta vul