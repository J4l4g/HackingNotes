
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.34 -oG allPorts`
`nmap -p22,80 -sCV 10.10.11.34 -oN targeted`
```python
22/ssh
80/http
```


`gobuster vhost -u http://trickster.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -r --append-domain`
```js
10.10.11.34     trickster.htb shop.trickster.htb
```

Utiliza un `prestashop` que es como un Wordpress para ecomerce

Encontramos en `robots.txt` el directorio `http://shop.trickster.htb/.git/`

Usaremos la herramienta [[GITHACK]] `https://github.com/lijiejie/GitHack.git`

