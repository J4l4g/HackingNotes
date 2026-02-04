
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.129.2.38 -oG allScan`

```
 [*] IP Address: 10.129.2.38
 [*] Open ports: 22,80,54321
 
```

`nmap -p 22,80,54321 -sCV 10.129.2.38 -oN targeted`


`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://facts.htb/FUZZ`
`http://facts.htb/admin/login`

Creamos una cuenta y accedemos con usuario creado, nos encontramos con un panel de admin estilo wordpress
