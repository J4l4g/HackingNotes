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

