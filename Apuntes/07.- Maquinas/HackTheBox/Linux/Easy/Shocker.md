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

Al no encontrar ninguna coincidencia puede ser que el servidor este rechazando asi que incluiremos una `/` al final del `FUZZ`
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.55.91/FUZZ/
```

Encontrando correspondencias como: `cgi-bin`, `icons` y `server-status`con código de estado *403*
Al ver `cgi-bin` buscaremos en Internet que es, y descubrimos que es una carpeta que se encuentra en un servidor web con la capacidad de almacenar scripts *CGI (Common Gateway Interface* ejecutables.




