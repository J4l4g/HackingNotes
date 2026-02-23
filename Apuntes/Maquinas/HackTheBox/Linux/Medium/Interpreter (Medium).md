

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.3.105 -oG allPorts
```

```shell
[*] IP Address: 10.129.3.105
[*] Open ports: 22,80,443,6661
```

```shell
nmap -p22,80,443,6661 -sCV 10.129.3.105 -oN targeted
```

Navegamos al puerto `80`

