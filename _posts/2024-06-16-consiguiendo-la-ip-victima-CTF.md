---
title: "Averigua la IP de tu Máquina Víctima en CTF"
date: 2024-06-16 19:00:00 +0200
categories: [CTF, Tutoriales]
tags: [CTF, Tutoriales]
image: /img/posts/ip-adress.png
---

En los retos de Capture The Flag (CTF), es fundamental conocer cómo identificar la dirección IP de nuestra máquina víctima. Este paso es crucial para aplicar correctamente nuestras técnicas y estrategias en el entorno virtual. En este tutorial, aprenderemos los métodos más efectivos para averiguar la IP de nuestra máquina víctima en un entorno CTF. Vamos a ver cómo hacerlo de manera práctica y eficiente.

## Inicio de CTF

Generalmente virtualizamos la máquina víctima que debemos vulnerar. Esto se puede hacer utilizando cualquier software de virtualización como VMware, VirtualBox, Docker, entre otros. En estos casos, conocemos la dirección IP de la máquina víctima.

Sin embargo, pueden darse situaciones en las que la IP esté camuflada o la máquina sea un sistema Windows que no visualicemos directamente. En estos casos, es crucial saber cómo descubrir la IP de nuestra máquina objetivo.

### ¿Por qué necesitamos saber la IP si siempre nos la facilitan?

Es importante mejorar con cada paso que damos. Esta práctica no solo es esencial en el mundo de la ciberseguridad, sino en cualquier aspecto de nuestra vida. Repetir un código, una ruta, o una variable sin entenderlos, simplemente copiando y pegando, no nos ayudará a aprender realmente.

Escribir y entender estos elementos nos permitirá aprender de verdad y mejorar nuestras habilidades. Por eso, esta guía es fundamental. Vamos a ver cómo identificar la IP de nuestra máquina víctima de manera efectiva y práctica.

## Identificando IP Maquina Victima

Si utilizamos VMware o VirtualBox, es recomendable crear una  [RED NAT](#configuracion-red-nat)para trabajar en un rango distinto al de nuestra red doméstica y no interferir con otros dispositivos. Una vez que sabemos qué red hemos asignado, podemos comenzar a identificar la IP de nuestra máquina víctima utilizando dos herramientas: [NMAP](#nmap) y [ARP-SCAN](#arp-scan).

### Configuración de la RED NAT {#configuracion-red-nat}

Configurar una RED NAT permite que nuestras máquinas virtuales tengan acceso a la red externa, mientras que otras máquinas en nuestra red local no pueden acceder a ellas. Esto proporciona una capa adicional de seguridad y organización. Asegúrate de configurar la red NAT adecuadamente en tu software de virtualización antes de proceder.


### NMAP {#nmap}

NMAP es una poderosa herramienta de escaneo de red que nos permite descubrir dispositivos conectados a nuestra red, así como obtener información detallada sobre ellos. Para identificar la IP de nuestra máquina víctima con NMAP, podemos utilizar el siguiente comando:
```bash
nmap -sn 172.17.0.1/24
```

Este comando realiza un escaneo "ping" en el rango de IP especificado (172.17.0.1/24), buscando dispositivos activos. La opción -sn indica un escaneo sin puertos, lo que significa que solo se busca si los hosts están activos sin escanear los puertos.

Aquí está un ejemplo del resultado de este comando:

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2024-06-16 19:41 CEST
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.00026s latency).
MAC Address: 00:50:56:F7:07:DE (VMware)
Nmap scan report for 172.17.0.145
Host is up (0.00027s latency).
MAC Address: 00:0C:29:BF:B0:38 (VMware)
Nmap scan report for 172.17.0.254
Host is up (0.00021s latency).
MAC Address: 00:50:56:E9:68:4F (VMware)
Nmap scan report for 172.17.0.1
Host is up.
Nmap scan report for 172.17.0.140
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 1.99 seconds
```

**MAC Address: 00:0C:29:BF:B0:38 (VMware): Dirección MAC del dispositivo.**

La dirección MAC **00:0C:29:BF:B0:38** es una pista crucial. Los primeros dos octetos de la dirección MAC **(00:0C)** corresponden al fabricante VMware. Esto nos indica que este dispositivo es una máquina virtual creada en VMware. Dado que estamos buscando la máquina víctima virtualizada y conocemos que estamos utilizando VMware para este propósito, podemos concluir que la IP **172.17.0.145** corresponde a nuestra máquina víctima.

**Nota:** También es posible que una dirección MAC que comience con 08:00 pertenezca a una máquina virtual.


### ARP-SCAN

ARP-SCAN es una herramienta útil para identificar dispositivos en la red local. Funciona enviando paquetes ARP (Address Resolution Protocol) a todas las direcciones IP en un rango específico y recopilando las respuestas. Es especialmente eficaz para descubrir rápidamente todos los dispositivos en la red local.

Para utilizar ARP-SCAN, ejecuta el siguiente comando especificando la interfaz de red:

```bash
arp-scan -I ens33 --localnet
Interface: ens33, type: EN10MB, MAC: 00:0c:29:63:18:d7, IPv4: 172.17.0.140
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
172.17.0.1	00:50:56:c0:00:08	VMware, Inc.
172.17.0.2	00:50:56:f7:07:de	VMware, Inc.
172.17.0.145	00:0c:29:bf:b0:38	VMware, Inc.
172.17.0.254	00:50:56:e9:68:4f	VMware, Inc.

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.131 seconds (120.13 hosts/sec). 4 responded
```
Entre estos resultados, hemos identificado que la IP **172.17.0.145** tiene la dirección MAC **00:0c:29:bf:b0:38**. Los primeros dos octetos de la dirección MAC **00:0c** son asignados a VMware, lo que nos indica que este dispositivo es una máquina virtual creada en VMware.

Dado que estamos buscando la máquina víctima virtualizada y conocemos que estamos utilizando VMware para este propósito, podemos concluir que la IP 172.17.0.145 corresponde a nuestra máquina víctima.

**Nota:** También es posible que una dirección MAC que comience con 08:00 pertenezca a una máquina virtual.

## Resultado

Con esta información, hemos identificado de manera precisa la IP de nuestra máquina víctima virtualizada **172.17.0.145** con MAC **00:0C:29:BF:B0:38**. Ahora podemos usar esta información para avanzar en nuestro desafío de CTF.