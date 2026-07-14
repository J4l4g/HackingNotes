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

## Recopilacion de informacion con SharpHound
![[Pasted image 20260713234544.png]]

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

## COnexion con studvm
Nos conectamos con la maquina usando winrs
![[Pasted image 20260714081418.png]]

Cargamos las herramientas requeridas
![[Pasted image 20260714081502.png]]

