
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.129.2.38 -oG allScan`

```
 [*] IP Address: 10.129.2.38
 [*] Open ports: 22,80,54321
 
```

`nmap -p 22,80,54321 -sCV 10.129.2.38 -oN targeted`


`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://facts.htb/FUZZ`
`http://facts.htb/admin/login`

Creamos una cuenta y accedemos con usuario creado, nos encontramos con un panel de admin de camalen CMS version 2.9.0
Encontramos un campo de subida de ficheros a la hora de editar nuestro perfil

encontramos un exploit en github que nos lo automatiza 
`https://github.com/predyy/CVE-2025-2304/blob/main/exp.py`

Una vez tenemos el acceso como administrador empezamos a hcer reconocimiento de la interfaz de administrador donde encontramos Settings ->