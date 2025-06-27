# https://192.168.5.110/

#### Reconocimiento tecnologías web:
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
- https://192.168.5.110/console