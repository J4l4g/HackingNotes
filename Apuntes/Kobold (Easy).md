
```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.16.146 -oG allPorts
```

```shell
 [*] IP Address: 10.129.16.146
 [*] Open ports: 22,80,443,3552
```

```shell
nmap -p22,80,443,3552 -sCV -vvv 10.129.16.146 -oN targeted
```



