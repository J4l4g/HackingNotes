```shell
subfinder -dL <domain.txt> -all -recursive -o <output.txt>
```

```shell
cat <domain.txt> | httpx -sc -location -title -o <output.txt>
```

Este es para Nuclei
```shell
cat <output.txt> | awk '{print $1}' > <output_nu.txt>
```

>Utilizar IA para crear plantillas para nuclei

```shell
nuclei -list <output_nu.txt -t <plantillas>
```


