Antes de empezar la maquina se nos sumistran unas credenciales
```ad-info
 rose::KxEPkKe6R8su
```
# Enumeración

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.232.128 -oG allPorts
```

```shell
 [*] IP Address: 10.129.232.128
 [*] Open ports: 53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810
```

```shell
nmap -p53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810 -sCV 10.129.232.128 -oN targeted
```



## Enumeración del SMB

```shell
nxc smb 10.129.232.128
```

Añadimos al `/etc/hosts` la IP y el dominio `10.129.232.128  sequel.htb dc01 dc01.sequel.htb`



