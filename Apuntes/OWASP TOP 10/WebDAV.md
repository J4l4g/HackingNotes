#WebDAV #OWASP #Enumeracion #Explotacion 

#### Definición
>**WebDAV** (**Web Distributed Authoring and Versioning**) es una extensión del protocolo HTTP que permite a los usuarios **acceder** y **manipular** **archivos** en un servidor web a través de una conexión segura.

## Enumeración
Usaremos [[WHATWEB]] para identificar que tipo de tecnologías usa la pagina web es y así poder ver si es un WebDAV:
`whatweb <URL_Victima>`

Podemos usar una herramienta llamada [[DAVTEST]] nos enumera todos los tipos de archivos que se pueden subir a la web y cuales son ejecutables/interpretables, pero para ellos debemos de saber las credenciales
`davtest -url http://localhost/ -auth `