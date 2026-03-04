
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.9.43 -oG allPorts
```

```shell
[*] IP Address: 10.129.9.43
[*] Open ports: 80,135,445,5985
```

```shell
nmap -p80,135,445,5985 -sCV 10.129.9.43 -oN targeted
```


```shell
nxc smb 10.129.9.43
```

## HTTP
Accedemos a la IP a traves del navegador y probamos a acceder usando `admin::admim` 
![[Pasted image 20260304110713.png]]