`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.130 -oG allPorts`

```ad-done
80/tcp open  http    syn-ack ttl 63
```

`nmap -p80 -sCV 10.10.11.130 -oN targeted`

