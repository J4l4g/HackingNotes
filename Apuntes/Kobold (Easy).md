
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

Hacemos fuzzing sobre la web en búsqueda de directorios
```shell
ffuf -c -u https://kobold.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```

Al no encontrar nada haremos uno de subdominios
```shell
ffuf -c -u https://kobold.htb -H "Host: FUZZ.kobold.htb" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -fc 302
```

Encontrando los subdominios `mcp` y `bin`, los añadiremos al `/etc/hosts`
Navegaremos a `mcp`
