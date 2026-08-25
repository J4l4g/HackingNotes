```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.49.90 -oG allPorts
```

```shell
nmap -p22,80 -sCV 10.129.49.90 -oN targeted  
```

Usaremos whatweb para ver que tecnologias utiliza
```shell
whatweb http://10.129.49.90
```

No encontramos mucha informacion
Ejecutaremso el script de nmap `http-enum` que hace la funcion de fuzer
