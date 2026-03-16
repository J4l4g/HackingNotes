
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.10.76 -oG allPorts
```

```shell
  [*] IP Address: 10.129.10.76
  [*] Open ports: 22,80
```

```shell
nmap -p 22,80 -sCV -oN targeted 10.129.10.76
```

## HTTP

```shell
whatweb http://variatype.htb/
```

![[Pasted image 20260316130529.png]]

```shell
ffuf -c -u http://variatype.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```


