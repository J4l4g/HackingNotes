### Enumeración de puertos y servicios
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.10.11.214 -oG allPorts`
 ```ad-info
 PORT      STATE SERVICE
22/tcp    open  ssh     
50051/tcp open  unknown 
 ```

`nmap -p22,50051 -sCV 10.10.11.214 -oN targeted`

Observamos en el puerto **50051** el servicio `GRPC` 

### GRPC

> Este servicio tiene la función de  framework de comunicación de alto rendimiento, de código abierto, creado por Google, que permite que aplicaciones escritas en diferentes lenguajes se comuniquen entre sí de manera eficiente, basándose en el concepto de **RPC (Remote Procedure Call)** para invocar funciones remotas como si fueran locales

