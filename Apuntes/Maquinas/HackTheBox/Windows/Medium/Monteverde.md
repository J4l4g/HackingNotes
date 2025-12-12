### Enumeración

`nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.10.172 -oG allPorts`

`nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696,49749 -sCV 10.10.10.172 -oN targeted`

