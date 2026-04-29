## Tareas
- [ ]  Identifique una maquina en el dominio de destino donde haya una sesión de administrador de dominio disponible
- [ ] Comprometer la maquina y elevar privilegios a Administrador de dominio abusando de una reverse shell en dcorp-ci
- [ ] Escalar privilegios a Domain Admin abusando de la administración local derivada a través de dcorp-adminsrv. En dcorp-adminsrv listar la lista de permisos de la aplicaion
	- [ ] Lagunas en las reglas de Applocker
	- [ ] Deshabilitar Applocker modificando la GPO aplicable

**Flag 10: Proceso que usa svcadmin como cuenta de servicio en dcorp-mgmt**
**Flag 11: Hash NTL de la cuenta scvadmin en dcorp-mgmt**
**Flag 12: Intentamos extraer credenciales en texto plano para tareas programas el valor de la flag es similar a lsass, registry, credential vault, etc**
**Flag 13: Hash NTLM de srvadmin extraído de dcorp-adminsrv**
**Flag 14: Hash NTLM de websvc extraído de dcorp-adminsrv**
**Flag 15: Hash NTLM de appadmin extraído de dcorp-adminsrv**



