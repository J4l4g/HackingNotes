
```shell
nmap -p- --open --min-rate 5000 -n -Pn -sS -vvv 10.129.8.117 -oG allPorts
```

```shell
[*] IP Address: 10.129.8.117
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405
```

```shell
nmap -p53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405 -sCV 10.129.8.117 -oN targeted
```

```shell
nxc smb 10.129.8.117
```

```shell
[*]  x64 (name:DC1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
```

