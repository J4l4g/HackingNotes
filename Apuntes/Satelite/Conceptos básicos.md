
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
- Bandas de frecuencia (L^[1 - 2 GHz], S^[2 - 4 GHz], X^[8 - 12 GHz], Ku, Ka)