`nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.12.215 -oG allTarget`

```

[*] IP Address: 10.129.12.215
[*] Open ports: 80,1433,5985

```


`nmap -p 80,1433,5985 -sCV 10.129.12.215 -oN targeted`

Navegamos a la web y vemos que podemos crearnos una cuenta, mientras tanto hacemos fuzzing con 

`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://eighteen.htb/FUZZ`

Con la cuenta creada vemos varios campos donde podemos introducir algún tipo de dato

