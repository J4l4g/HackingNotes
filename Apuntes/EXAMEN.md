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


| **studvm$**      | `01766386a8a082a359e9baf437cc7ce6` | **Ataque S4U (Delegación)** - Este es el que necesitas para el RBCD |
| ---------------- | ---------------------------------- | ------------------------------------------------------------------- |
| **studentuser**  | `5bade58465eca3499257e8b90d99f9aa` | Movimiento lateral como tu usuario actual                           |
| **studentadmin** | `97daeac345542c952eea4446471ca158` | Acceso local a la máquina `studvm`                                  |

## GenericWrite sobre MGMTSRV


Captura BloodHound

![[Pasted image 20260714120401.png]]
Importamos powermad
```shell
. .\Powermad.ps1
```

![[Pasted image 20260714122827.png]]

![[Pasted image 20260714121726.png]]


