#Vulnhub

La maquina no responde a ping

###### Reconocimiento
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 192.168.1.26 -oG allPorts`

`whatweb http://192.168.1.26`
```python
http://192.168.1.26 [200 OK] Apache[2.4.18], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[192.168.1.26], JQuery[1.10.2], Modernizr[2.6.2.min], Script, Title[IMF - Homepage], X-UA-Compatible[IE=edge]
```

`nmap -p80,7788 -sCV 192.168.1.26 -oN targeted`
```python
# Nmap 7.95 scan initiated Wed Aug 27 21:59:46 2025 as: /usr/lib/nmap/nmap -p80 -sCV -oN targeted 192.168.1.26
Nmap scan report for 192.168.1.26
Host is up (0.00066s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: IMF - Homepage
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 08:00:27:02:65:07 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 27 21:59:58 2025 -- 1 IP address (1 host up) scanned in 11.50 seconds
```

`nmap --script http-enum -p80 192.168.1.26 -oN webScan`


#### En el navegador:

`192.168.1.26`
