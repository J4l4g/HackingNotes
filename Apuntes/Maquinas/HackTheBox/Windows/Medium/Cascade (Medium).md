
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


## 