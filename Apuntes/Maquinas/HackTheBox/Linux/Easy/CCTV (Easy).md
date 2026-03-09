
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.4.75 -oG allPorts
```

```shell
[*] IP Address: 10.129.4.75
[*] Open ports: 22,80
```

```shell
nmap -p22,80 -sCV 10.129.4.75 -oN targeted
```

