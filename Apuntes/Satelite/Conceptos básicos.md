
# Arquitectura de un sistema satelital
![[Pasted image 20260728181953.png]]

Un sistema satelital básico consta de tres segmentos principales básicos:
## 1. Segmento Espacial / Space Segment
Este segmento incluye el propio satélite y los siguientes componentes:
- Ordenador de abordo
- Sensores y payload
- Firmware / RTOS
- Sistema de potencia y control

Esta parte esta altamente protegida y es difícil de acceder a ella directamente.

## 2. Segmento Terrestre / Ground Segment
En este punto es donde se ejerce el control real, incluyendo:
- Estaciones terrestres
- Servidores de control de misión
- Sistemas de seguimiento
- Sistema de enlace ascendente de comandos

La mayoría de los ataques reales comienzan desde este punto.

## Segmento de Comunicación / Communication Segment
Este segmento es el que conecta la tierra con el espacio aéreo, incluyendo:
- Enlace ascendente de RF^[Radiofrecuencia]
- Enlace descendente de RF
- Flujo de telemetría
- Canal de comandos
- Capa de cifrado

Una debilidad en esta capa puede exponer datos confidenciales.

# Superficie de ataque de los sistemas satelitales
Los puntos débiles que incluyen estos sistemas satelitales son varios, entre ellos:
- Infraestructura terrestre
- Capa de comunicación RF
- Sistema de control TT&C
- Firmware /Software integrado
- Servicios en la nube / API / Red
- Cadena de suministro

A la hora de hacer una investigación profesional en estos entornos se centra en estas capas en lugar de solo en el satélite.

## Capa de comunicación por radiofrecuencia
**Invisible pero explotable**

![[Pasted image 20260728183922.png]]

En este punto es donde viajan las señales entre la Tierra y el espacio, esto incluye:
- Enlace ascendente (Comandos enviados al satélite)
- Enlace descendente (Telemetría / datos recibidos)
- Bandas de frecuencia (L^[1-2 GHz], S^[2-4 GHz], X^[8-12 GHz], Ku^[12-18 GHz], Ka^[26-40 GHz])

>**Usos de las bandas**
>*- L ->* GPS/GNSS, comunicaciones móviles satelitales (Iridium, Inmarsat), telemetría de algunos CubeSats
>*- S ->* Telemetría, seguimiento y comando (TT&C) de la mayoría de satélites — es la banda "de servicio" por excelencia para controlar el satélite, no para la misión principal
>*- X ->* Comunicaciones militares, radar, y transmisión de datos científicos/imágenes de observación terrestre (muchos satélites de imagen usan X-band para bajar datos a alta velocidad)
>*- Ku ->* TV satelital directa al hogar (DTH), VSAT, algunos servicios de internet satelital
>*- Ka ->* Internet satelital de alta velocidad (Starlink usa principalmente Ku y Ka), comunicaciones militares de alta capacidad, algunas misiones científicas de la NASA

Estas señales son tan arriesgadas de usar por que viajan a través del espacio abierto, pueden ser interceptados mediante SDR (Radio Definida por Software), el cifrado no siempre esta implementado correctamente.

## Campos de posible ataque
- Intercepción de señales (Escucha clandestina)
- Ataques de repetición
- Interferencia (Denegación de servicio)
- Suplantación de señales
-
```ad-hint
No necesitas acceso físico: la radiofrecuencia proporciona  alos atacantes una superficie de ataque remota.
```

# Sistemas TT&C (Telemetría, Seguimiento y Comando)
![[Pasted image 20260728190153.png]]

Este es canal de comunicación cerebral del satélite, incluyendo:
- Datos de telemetría (Salud / Estado)
- Datos de seguimiento (Posición / Órbita)
- Ejecución de comandos.

Es una pieza fundamental ya que controla directamente el comportamiento de satélite, cualquier compromiso equivale a un impacto total en el sistema.

## Vectores de ataque
- Inyección de comandos no autorizado
- Manipulación de telemetría
- Comando de repetición

```ad-hint
Si el sistema de TT&C se ve comprometido, el atacante no hackea, sino que se convierte en el operador.
```

# Firmware de sistemas embebidos
![[Pasted image 20260728192757.png]]

Los satélites funcionan con sistemas integrados altamente especializados, estos sistemas incluyen:
- RTOS (Sistemas Operativos en Tiempo Real)
- Software de vuelo
- Interfaces de hardware

Es arriesgado debido a que es difícil de parchear una vez integrado, tiene un ciclo de vida muy prolongado ( 10-15 años o mas) y además a menudo es construido con componentes antiguos

## Vectores de ataque
- Ingeniería inversa de firmware
- Puertas traseras en los mecanismos de actualización
- Explotar fallos de memoria lógica

```ad-hint
Una única vulnerabilidad de firmware puede persistir durante años en órbita
```


# Nube, API y software terrestre
![[Pasted image 20260728193605.png]]

