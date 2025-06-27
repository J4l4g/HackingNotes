# https://192.168.5.110/

#### Reconocimiento 
*`nmap -p- --open --min-rate 1000 -sSCV -n -Pn -vvv 192.168.5.110 -oG allPorts`*

Hallazgos:
```ruby
PORT      STATE SERVICE      REASON
22/tcp    open  ssh          syn-ack ttl 62
443/tcp   open  https        syn-ack ttl 62
2000/tcp  open  cisco-sccp   syn-ack ttl 64
5060/tcp  open  sip          syn-ack ttl 64
10050/tcp open  zabbix-agent syn-ack ttl 62
```

#### Tecnologías web:
*`whatweb https://192.168.5.110/`*

```ruby
https://192.168.5.110/ [200 OK] Bootstrap, 
Cookies[JSESSIONID], 
Country[RESERVED][ZZ], 
HTML5, HttpOnly[JSESSIONID], 
IP[192.168.5.110], 
JQuery[3.6.1], 
Java[2.3], PasswordField[J_PASSWORD,anterior,nueva,nuevarep], Script[module,text/javascript], 
Strict-Transport-Security[max-age=31536000; includeSubDomains], 
Title[HP-HCIS Control d`accés Node:hcis4pre01], UncommonHeaders[content-security-policy-report-only], X-Frame-Options[SAMEORIGIN], X-Powered-By[JSP/2.3], X-XSS-Protection[1; mode=block]

```

*`wapapalyzer`*

![[Pasted image 20250627100447.png]]

#### Login
*Inyección SQL `sqlmap`*
`sqlmap -u https://192.168.5.110/ --forms --dbs --batch`

*Fuzzing `gobuster`*
`gobuster dir -u https://192.168.5.110/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -k`

Hallazgos
- https://192.168.5.110/console  CODE:302

*Fuzzing `wfuzz`*
` wfuzz -c --hc=404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "https://192.168.5.110/FUZZ" -t 20`

Hallazgos
- https://192.168.5.110/console  CODE: 302

*Interceptar petición login `burpsuite`*
```ruby
POST /hphis/ProcesarLogin.jsp HTTP/1.1
Host: 192.168.5.110
Cookie: JSESSIONID="VoI3sT0I6ZEu_6Aw_1rMkePXAc11gsObTFv9UEJt.Nodo1:hcis"; login_USERNAME="OLn3tAXO8jb5juQYgxOx2huS7bPRZZZe3p+fW876lQGXyp5tlds6jA=="
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://192.168.5.110/
Content-Type: application/x-www-form-urlencoded
Content-Length: 183
Origin: https://192.168.5.110
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

_csrf=7a46eafd-1c66-4459-82ac-ed724e10c04e&url=&sil=HNSM&centro=HNSM&soloLogueo=N&timestamp=1751014109671&J_USERNAME=odelpozo&J_PASSWORD=SAAS000&Conexion=HNSM%7CHNSM&unidad=Sin+Unidad
```

Hallazgos:
- Se tramitan las credenciales en texto claro

## PAGINA PRINCIPAL https://192.168.5.110/hphis/edoctor/principal.jsp



