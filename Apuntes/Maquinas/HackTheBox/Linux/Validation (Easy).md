## Reconocimiento

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.116 -oG allPorts`

`nmap -p 22,80,4566,8080 -sCV -vvv 10.10.11.116 -oN targeted`

```
22/ssh
80/http
4566/http
8080/http
```