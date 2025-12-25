### Enumeración de puertos y servicios
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.10.11.214 -oG allPorts`
 ```ad-info
 PORT      STATE SERVICE
22/tcp    open  ssh     
50051/tcp open  unknown 
 ```

`nmap -p22,50051 -sCV 10.10.11.214 -oN targeted`
