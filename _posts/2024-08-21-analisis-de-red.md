---
title: Análisis de IPs sospechosas en red con Bash y AbuseIPDB
date: 2024-08-20 19:2:00 +0200
categories: [GitHub, Scripting]
tags: [Seguridad, Redes, Bash, AbuseIPDB, Análisis de Red]
image: /img/posts/logo-ip.png
---

## Introducción

En entornos de seguridad informática, es crucial detectar y analizar tráfico sospechoso en la red. Este post detalla un enfoque automatizado para identificar y verificar IPs sospechosas en capturas de red (PCAP), utilizando scripts en Bash y la API de [AbuseIPDB](https://www.abuseipdb.com/). Esta metodología es útil para analistas de seguridad que necesitan revisar grandes volúmenes de datos y priorizar la investigación en IPs con mayor probabilidad de estar asociadas con actividades maliciosas.

## ¿Qué es AbuseIPDB?

[AbuseIPDB](https://www.abuseipdb.com/) es un servicio colaborativo que permite a los usuarios reportar direcciones IP involucradas en actividades maliciosas. Con su API, es posible consultar el nivel de "confianza de abuso" de una IP, lo que indica la probabilidad de que dicha IP esté asociada con comportamientos maliciosos. Esto es especialmente útil cuando nosotros analizamos registros de red para identificar amenazas potenciales.

## Captura de tráfico de red

Antes de ejecutar el script de análisis y verificación de IPs, primero necesitamos capturar el tráfico de red en un archivo .pcapng. Para esto, utilizaremos la herramienta `tshark`, que es la versión en línea de comandos de Wireshark.

El siguiente comando captura el tráfico de red en la interfaz especificada y lo guarda en un archivo llamado `captura.pcapng`:

```bash
tshark -i ens33 -w captura.pcapng
```

### Explicación del comando:

- `-i ens33`: Especifica la interfaz de red que vamos a monitorear. `ens33` es un nombre de interfaz de red típico en sistemas Linux, pero este puede variar dependiendo de la configuración de tu sistema.

- `-w captura.pcapng`: Indica que el tráfico capturado se guardará en un archivo llamado `captura.pcapng`.

### ¿Cómo identificar tu interfaz de red?

Para encontrar el nombre de tu interfaz de red en un sistema Linux, puedes usar el comando `ip a` o `ifconfig`. Estos comandos listarán todas las interfaces de red disponibles junto con sus nombres. El nombre de la interfaz será necesario para ejecutar `tshark` correctamente. Por ejemplo:

```bash
ip a
```

Una vez que tengamos el archivo `captura.pcapng`, estaremos listos para ejecutar el script que analizará este archivo, identificará IPs sospechosas, y las verificará contra la base de datos de AbuseIPDB.

## Objetivo del Script

El objetivo principal de este script es automatizar todo el proceso, desde el análisis de la captura de red hasta la verificación de direcciones IP sospechosas. Esto incluye la identificación de IPs que envían paquetes SYN sin recibir ACK (un patrón típico en ataques de escaneo de puertos), el filtrado de IPs externas (no locales), y la verificación de estas IPs contra la base de datos de AbuseIPDB para determinar si han sido reportadas por actividades maliciosas.

## El Script Completo

Una vez que tengamos el archivo `captura.pcapng`, podemos ejecutar el siguiente script para llevar a cabo todo el proceso de análisis y verificación.

```bash
#!/bin/bash

tshark -r captura.pcapng -T fields -e ip.src -e ip.dst -e tcp.flags -Y "tcp.flags.syn == 1 and tcp.flags.ack == 0" > ips_sospechosas.txt

# El comando `tshark -r captura.pcapng` lee el archivo de captura de red.
# `-T fields` indica que queremos extraer campos específicos.
# `-e ip.src -e ip.dst -e tcp.flags` especifica los campos a extraer: IP de origen, IP de destino y flags TCP.
# `-Y "tcp.flags.syn == 1 and tcp.flags.ack == 0"` es un filtro que selecciona paquetes TCP con el flag SYN establecido y el flag ACK no establecido (escaneo de puertos).
# La salida se guarda en `ips_sospechosas.txt`.


grep -vE "^192\.168\." ips_sospechosas.txt > ips_sospechosas_filtradas.txt

# `grep -vE "^192\.168\."` filtra las IPs que comienzan con `192.168.` (una red local típica).
# `ips_sospechosas.txt` es el archivo de entrada y `ips_sospechosas_filtradas.txt` es el archivo de salida con las IPs locales eliminadas.


API_KEY="YOUR_ABUSEIPDB_API_KEY"
INPUT_FILE="ips_sospechosas_filtradas.txt"
OUTPUT_VERIFIED_CSV="verificadas_ips.csv"
> $OUTPUT_VERIFIED_CSV

# `API_KEY` debe ser reemplazada con tu clave API de AbuseIPDB para autenticar las solicitudes.
# `INPUT_FILE` es el archivo con IPs filtradas para verificar.
# `OUTPUT_VERIFIED_CSV` es el archivo donde se almacenarán los resultados de la verificación.

while IFS=$'\t' read -r src_ip dst_ip flags; do
    response=$(curl -s "https://api.abuseipdb.com/api/v2/check?ipAddress=$src_ip&maxAgeInDays=90" \
    -H "Key: $API_KEY" \
    -H "Accept: application/json")

    is_abused=$(echo "$response" | jq -r '.data.abuseConfidenceScore')

    if [[ "$is_abused" != "null" && "$is_abused" -ge 50 ]]; then
        echo -e "$src_ip\t$dst_ip\t$flags\tAbused\t$is_abused" >> $OUTPUT_VERIFIED_CSV
    else
        echo -e "$src_ip\t$dst_ip\t$flags\tNot Abused\t$is_abused" >> $OUTPUT_VERIFIED_CSV
    fi

    sleep 1  
done < "$INPUT_FILE"

# El bucle `while` lee el archivo de IPs filtradas línea por línea.
# `curl` envía una solicitud a la API de AbuseIPDB para verificar la IP de origen.
# `jq` extrae el puntaje de confianza de abuso de la respuesta JSON. (NOTA, se requiere de jq, si no lo tenemos, instalar )
# Si el puntaje es mayor o igual a 50, se considera que la IP está reportada por abuso.
# Los resultados se guardan en `verificadas_ips.csv`.
# `sleep 1` se utiliza para evitar exceder el límite de solicitudes de la API.

echo "IPs verificadas han sido registradas en $OUTPUT_VERIFIED_CSV"

# El mensaje de echo informa al usuario que el proceso ha terminado y que el archivo con las IPs verificadas está disponible.

```
## Resultado

```bash
sudo ./analizar_ips.sh 
Running as user "root" and group "root". This could be dangerous.
IPs verificadas han sido registradas en verificadas_ips.csv

cat verificadas_ips.csv 

116.59.29.87	192.168.1.38	0x0002	Abused	99
65.49.1.59	    192.168.1.38	0x0002	Abused	100
152.42.253.17	192.168.1.38	0x0002	Not Abused	28
104.209.34.218	192.168.1.38	0x0002	Abused	100
93.174.93.12	192.168.1.38	0x0002	Abused	100
111.23.122.227	192.168.1.38	0x0002	Abused	100
95.214.55.138	192.168.1.38	0x0002	Abused	100
172.169.207.62	192.168.1.38	0x0002	Abused	100
141.255.160.234	192.168.1.38	0x0002	Abused	100
93.174.93.12	192.168.1.38	0x0002	Abused	100
185.224.128.83	192.168.1.38	0x0002	Abused	100
154.213.184.25	192.168.1.38	0x0002	Abused	100
205.210.31.231	192.168.1.38	0x0002	Not Abused	0
149.50.103.48	192.168.1.38	0x0002	Abused	100
62.169.23.115	192.168.1.38	0x0002	Abused	100
```

## Conclusión

Este script es una herramienta poderosa para cualquier analista de seguridad que necesite procesar y verificar IPs sospechosas de manera eficiente. Automáticamente, captura el tráfico de red, analiza la captura, filtra las IPs relevantes, y verifica su reputación en AbuseIPDB. Esto nos permite ahorrar tiempo y focalizarnos en IPs con mayor potencial de amenaza, mejorando así la eficiencia en la respuesta ante incidentes de seguridad.

Al seguir esta metodología, podemos integrar fácilmente la verificación de IPs en nuestros flujos de trabajo de análisis de red, proporcionando una capa adicional de seguridad en la detección y prevención de ataques en nuestra red.

¡Pruébalo y asegura tu red de manera proactiva!


Saludos, Óscar