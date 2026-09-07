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

No encontramos nada relevante, seguirermos haciendo fuizzing con otras herramintas
