Hacemos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] general para ver todo lo que encontráramos
`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.13 -oG AllPorts`

Mas adelante hacemos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia los puertos que hemos descubierto abiertos
`nmap -p 80,3306,33060 --open -sCV --min-rate 5000 -vvv -n -Pn 192.168.1.13 -oN ScanPorts`

Empezamos investigando por el puerto 80
Usamos Whatweb para ver que tecnologías corren en el servidor web
`whatweb http://192.168.1.13/`
![[Pasted image 20241227114509.png]]

No encontramos nada relevante para empezare así que comenzaremos investigando directamente en la web
Buscando en el código fuente encontramos los siguientes enlaces
![[Pasted image 20241227115820.png]]

En `main.js` encontramos la siguiente fuga de información en cuanto a un direcctorio qu enos lleva  aun panel de login
![[Pasted image 20241227115910.png]]

![[Pasted image 20241227115949.png]]

Buscamos en internet información sobre SeedDMS y encontramos que es de código abierto y tienen un árbol de directorios en el cual se encuentra `/config/.htaccess` hemos encontrado esto en GitHub en el código por defecto de `.htaccess` nos aparece el siguiente comentario
![[Pasted image 20241227120356.png]]

Advirtiendo de que nos aseguremos de que el fichero `settings.xml` no se pueda leer desde fuera, vamos a navegar en la URL al fichero `http://192.168.1.13/seeddms51x/conf/settings.xml`
Vamos buscando encontrar credenciales, podemos usar la búsqueda del navegador para buscar por palabras. Encontramos los siguientes datos
![[Pasted image 20241227120609.png]]

Son el usuario y la password de acceso a la base de datos.
Como hemos visto en el principio en el escaneo estaba abierto el puerto 3306 que es el puerto de MySQL así que vamos a intentar acceder a la base de datos a través de las credenciales obtenidas.

