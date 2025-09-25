levi.james / KingofAkron2025!
## Reconocimiento
`rustscan -a $IP --ulimit 1000 -r 1-65535 -- -A -sCV -oG allPorts`

*Puertos*
`53/DNS`
`88/kerberos`
`111/rpcbind`
`389/LDAP`
`445/SMB`
`5985/WinRm`


`netexec ldap $IP 2>/dev/null` 
`LDAP 10.10.11.70 389 DC [*] Windows Server 2022 Build 20348 (name:DC) (domain:PUPPY.HTB)`

