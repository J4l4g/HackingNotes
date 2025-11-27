`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.130 -oG allPorts`

```ad-done
80/tcp open  http    syn-ack ttl 63
```

`nmap -p80 -sCV 10.10.11.130 -oN targeted`

```ad-done
80/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.9.2)
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
```

`whatweb http://10.10.11.130`

