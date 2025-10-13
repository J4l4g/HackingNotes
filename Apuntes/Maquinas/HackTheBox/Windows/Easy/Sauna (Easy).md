`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.10.175 -oG allPorts`
`nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49668,49673,49674,49677,49689,49697 -sCV 10.10.10.175 -oN targeted`
 ``` python
53/tcp    open  domain       
80/tcp    open  http         
|_http-server-header: Microso
|_http-title: Egotistical Ban
| http-methods: 
|_  Potentially risky methods
88/tcp    open  kerberos-sec 
135/tcp   open  msrpc        
139/tcp   open  netbios-ssn  
389/tcp   open  ldap         
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         
3269/tcp  open  tcpwrapped
5985/tcp  open  http         
|_http-server-header: Microso
|_http-title: Not Found
9389/tcp  open  mc-nmf       
49668/tcp open  msrpc        
49673/tcp open  ncacn_http   
49674/tcp open  msrpc        
49677/tcp open  msrpc        
49689/tcp open  msrpc        
49697/tcp open  msrpc        
 ```


#### 445
` netexec smb $IP `
