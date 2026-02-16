
# Reconocimineto

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.6.141 -oG allPorts
```

```sehll
 [*] IP Address: 10.129.6.141
 [*] Open ports: 22,80
```

```shell
nmap -p22,80 -sCV -vvv 10.129.6.141 -oN targeted
```

