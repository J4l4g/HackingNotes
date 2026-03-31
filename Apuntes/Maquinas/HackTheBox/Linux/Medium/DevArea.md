
```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.21.32 -oG allPorts
```

```shell
  [*] IP Address: 10.129.21.32
  [*] Open ports: 21,22,80,8080,8500,8888
```

```shell
nmap -p 21,22,80,8080,8500,8888 -sCV 10.129.21.32 -oN targeted
```

## FTP
```shell
ftp 10.129.21.32
```

## HTTP
```shell
whatweb http://devarea.htb/
```

```shell
ffuf -c -u http://devarea.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
```

