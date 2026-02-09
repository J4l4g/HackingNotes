
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.3.189 -oG allPorts`

```shell
[*] IP Address: 10.129.3.189
[*] Open ports: 22,80
```

`nmap -p22,80 -sCV 10.129.3.189 -oN targeted`

`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://pterodactyl.htb/FUZZ`

No encontramos nada, pero como vemos que habla de un subdominio `play` haremos una enumeracion de subdominios
`wfuzz -c --hc 404,302 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.pterodactyl.htb" http://pterodactyl.htb`

encontramos el subdominio `panel`
Encontrando un panel de login en la url `http://panel.pterodactyl.htb/auth/login`


