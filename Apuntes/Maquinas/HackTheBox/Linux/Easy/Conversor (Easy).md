
`nmap -p- --open -sS --min-rate 5000 -n -Pn 10.129.15.195 -oG allPorts`

``` shell
[*] IP Address: 10.129.15.195
[*] Open ports: 22,80
```

`nmap -p 22,80 -sCV 10.129.15.195 -oN targeted`

Encontamos una web con un panel de login y registro, hacemos un poco de fuzzing mientras investigamos la web en busqueda de algo quye nos pueda aportar mas informacion
`wfuzz --hc 404 -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://conversor.htb/FUZZ`

Al no encontrar nada nos creamos una cuneta y accedemos a una web en la que nso convierte archivos 