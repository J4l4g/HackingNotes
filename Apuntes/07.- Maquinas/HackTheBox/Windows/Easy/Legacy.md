```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.227.181 -oG allPorts
```

```shell
nmap -p135,139,445 -sCV -vvv 10.129.227.181 -oN targeted 
```

