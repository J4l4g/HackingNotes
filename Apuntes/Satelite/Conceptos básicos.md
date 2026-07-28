
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
- Enlace ascendente de RF^[Radio Frecuencia]
- Enlace descendente de RF
- Flujo de telemetría
- Canal de comandos
- Capa de cifrado

Una debilidad en esta capa puede

