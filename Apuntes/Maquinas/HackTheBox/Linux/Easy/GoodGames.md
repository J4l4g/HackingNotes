`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.130 -oG allPorts`

```ad-done
80/tcp open  http    syn-ack ttl 63
```

`nmap -p80 -sCV 10.10.11.130 -oN targeted`

```ad-done
80/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.9.2)
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
```

`whatweb http://10.10.11.130`

Encontramos un icono de un usuario para hacer login, junto con un panel de registro

Usaremos [[BURPSUITE]] para interceptar el trafico y poder manipularlo

Interceptamos le trafico del login con burposuite y le añadimos despues del email `' or 1=1-- -`
Al enviarlo obtenemos una respuesta en la que se intenta setear una cookie de sision
![[2025-11-27 13_00_48-KaliLinux [Corriendo] - Oracle VirtualBox.png]]

Y mas para abajo nos muestra
![[2025-11-27 15_15_33-KaliLinux [Corriendo] - Oracle VirtualBox.png]]

Asi que eso quiere decir que es vulnerable a una inyeccion sql

### SQL Injection

