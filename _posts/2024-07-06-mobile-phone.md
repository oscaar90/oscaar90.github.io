---
title: "Writeup: [Mobile Phone] - TheHackerLabs"
date: 2024-07-06 14:35:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Android, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Android, Principiante]
image: /img/posts/CTF/MobilePhone/mobile-phone.png
---

## Introducción

En esta oportunidad, nos adentraremos en el desafío **"Mobile Phone"**, un reto diseñado para principiantes que se ejecuta sobre un sistema operativo Android. Este CTF, creado por ``@CuriosidadesDeHackers y @condor7777``, nos ofrece una oportunidad única para explorar vulnerabilidades y técnicas de explotación en dispositivos móviles.

🌐 [**Web oficial del CTF: TheHackersLabs - Mobile Phone**](https://thehackerslabs.com/mobile-phone/){:target="_blank"}

"Mobile Phone" presenta una serie de desafíos enfocados en entender y explotar las configuraciones y aplicaciones comunes en Android, proporcionando una plataforma ideal para aquellos que están empezando en el mundo del hacking ético de dispositivos móviles. A través de este writeup, compartiré mis experiencias, las estrategias que empleé, y las lecciones aprendidas durante la resolución de este interesante CTF.

## Identificación de la Víctima con Arp-Scan

Antes de sumergirnos en el escaneo de puertos y la explotación, es crucial identificar correctamente el dispositivo objetivo dentro de la red local. Para esto, utilizamos `arp-scan`, una herramienta potente para descubrir dispositivos activos en la red local.

### Ejecución de Arp-Scan

Desde nuestra terminal, lanzamos un `arp-scan` en la interfaz de red para detectar todos los dispositivos conectados:

```bash
arp-scan -I eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:a8:9c:98, IPv4: 172.18.0.133
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
172.18.0.1                                 VMware, Inc.
172.18.0.2                                 VMware, Inc.
172.18.0.135    00:0c:29:d3:30:83          VMware, Inc.
172.18.0.254                               VMware, Inc.
4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.113 seconds (121.15 hosts/sec). 4 responded
```
Destacamos que la dirección IP 172.18.0.135 pertenece a un dispositivo de VMware, Inc., identificado como nuestro objetivo para el siguiente paso del análisis.

## Más sobre Arp-Scan
Para aquellos interesados en aprender más sobre cómo utilizar arp-scan para identificar dispositivos en una red, he escrito un post detallado que puedes encontrar aquí: 🌐 [**Consiguiendo la IP de la Víctima en CTFs**](/posts/consiguiendo-la-ip-victima-CTF/){:target="_blank"}. Este recurso proporciona guías adicionales y consejos prácticos para utilizar eficazmente esta herramienta en tus propios retos de CTF.

## Exploración de Puertos con Nmap

Para comenzar nuestro análisis, realizamos un escaneo exhaustivo de los puertos del dispositivo utilizando la herramienta `nmap`. Este escaneo nos permite identificar puertos abiertos y servicios activos que podrían revelar vulnerabilidades potenciales en el sistema operativo Android de la víctima.

### Comando de Nmap Utilizado

El comando que utilizamos para realizar el escaneo fue el siguiente:

1. -p-: Escanea todos los puertos.
2. --open: Muestra solo los puertos abiertos.
3. --min-rate 2000: Establece la tasa mínima de paquetes enviados por segundo a 2000
4. -sS: Realiza un escaneo SYN, que es rápido y menos intrusivo.
5. -Pn: Omite el descubrimiento de hosts (no ping).
5. -n: No resuelve nombres DNS.
5. -vvv: Aumenta la verbosidad para mostrar más detalles sobre el escaneo.
8. 172.18.0.135: Especifica la dirección IP del objetivo.
9. -oG allports_mobile: Guarda los resultados en un archivo con formato greppable.

```bash
nmap -p- --open --min-rate 2000 -sS -Pn -n -vvv 172.18.0.135 -oG allports_mobile
```


(Aquí adjuntarías la imagen del resultado del escaneo)

**Descubrimiento Importante: Puerto 5555**

El resultado del escaneo reveló que el puerto 5555 estaba abierto, conocido por ser utilizado por el ``Android Debug Bridge (ADB)``, una herramienta vital para la administración remota de dispositivos Android. Tras consultar diversas fuentes, encontramos un artículo detallado en HackTricks que fue particularmente útil:


🌐 [**HackTricks: Android Debug Bridge**](https://book.hacktricks.xyz/v/es/network-services-pentesting/5555-android-debug-bridge){:target="_blank"}


Este recurso nos ofreció información detallada y técnicas probadas para evaluar y explotar posibles debilidades relacionadas con este puerto. Nos guiará en los próximos pasos para conectar a través de ADB y explorar más a fondo las posibilidades de manipulación y control del dispositivo Android.



## Intento de Conexión con ADB y Configuración Necesaria

Después de revisar la información de HackTricks sobre el uso de ADB para explotar el puerto 5555, intentamos establecer una conexión utilizando el comando `adb connect`. Sin embargo, nos encontramos con un pequeño obstáculo.

### Problema al Conectar con ADB

Al intentar conectar con el dispositivo a través de ADB, nos dimos cuenta de que no teníamos ADB instalado en nuestra máquina. El sistema nos proporcionó las siguientes indicaciones para la instalación:

```bash
adb connect 172.18.0.135
Command 'adb' not found, but can be installed with:
apt install adb
apt install google-android-platform-tools-installer
```

### Instalación de ADB
Para resolver este inconveniente y proceder con nuestra exploración, decidimos instalar los paquetes necesarios. Esto nos permitiría interactuar adecuadamente con el dispositivo Android a través del puerto 5555:

```bash
sudo apt install adb
sudo apt install google-android-platform-tools-installer
```

Con ADB ahora instalado, podríamos proceder a establecer una conexión con el dispositivo.

**Consulta de Recursos Adicionales**

Además, buscamos información adicional para asegurarnos de que estábamos utilizando ADB correctamente. Encontramos recursos útiles en la página oficial de los Desarrolladores de Android, que ofrece guías y consejos detallados sobre cómo utilizar ADB eficazmente:


🌐 [**Android Developers - ADB**](https://developer.android.com/tools/adb?hl=es-419){:target="_blank"}


## Conexión Exitosa con ADB

Con ADB correctamente instalado en nuestro sistema, procedimos a seguir los pasos detallados en HackTricks para explotar el puerto abierto 5555 del dispositivo Android. Esto nos permitió establecer una conexión directa y manejar el dispositivo con mayor profundidad.

### Establecimiento de la Conexión

Al intentar conectar con el dispositivo mediante ADB, el sistema nos informó que el daemon no estaba en ejecución y procedió a iniciarlo automáticamente:

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/MobilePhone/adbconnect.png>) 


```bash
adb connect 172.18.0.135                               
* daemon not running; starting now at tcp:5037
* daemon started successfully
connected to 172.18.0.135:5555
```



## Obteniendo Acceso de Root
Una vez establecida la conexión, ejecutamos el comando adb root para reiniciar el servicio ADB con privilegios de root, lo que nos proporcionó un nivel de acceso superior sobre el dispositivo:

```bash
adb root                
restarting adbd as root
```
## Exploración del Sistema Android
Con privilegios de root confirmados, utilizamos adb shell para explorar el sistema de archivos del dispositivo:
```bash
adb shell
root@x86_64:/ # ls
acct
cache
charger
config
d
data
default.prop
dev
etc
file_contexts
fstab.android_x86_64
init
init.android_x86_64.rc
init.bluetooth.rc
vendor
```
## Descubrimiento de la Flag

Durante nuestra exploración, logramos ubicar y visualizar los contenidos críticos que demostraron nuestro éxito completo en el reto:

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/MobilePhone/flasgroot.png>) 

## Conclusión del Reto

Hemos llegado al final de nuestro desafío con "Mobile Phone", habiendo demostrado cómo la exploración meticulosa y el manejo estratégico de las herramientas pueden revelar y explotar vulnerabilidades en dispositivos Android.

### Herramientas y Técnicas Utilizadas

Durante este reto, empleamos una serie de herramientas y técnicas fundamentales para el éxito de nuestra exploración:

- **Arp-scan**: Utilizado para identificar dispositivos activos en la red.
- **Nmap**: Empleado para escanear puertos y detectar servicios activos en el dispositivo.
- **ADB (Android Debug Bridge)**: Fundamental para establecer una conexión directa y gestionar el dispositivo con privilegios.
- **Búsqueda de Información**: Consultamos recursos como HackTricks para entender y aplicar técnicas específicas de explotación para Android.

### Importancia de la Protección de Dispositivos

Este reto destaca la importancia de proteger nuestros dispositivos contra accesos no autorizados:

- **Actualizaciones Regulares**: Es crucial mantener el software del dispositivo actualizado para protegerse contra vulnerabilidades conocidas.
- **Configuraciones de Seguridad Adecuadas**: Configura correctamente las opciones de seguridad en tus dispositivos, deshabilitando servicios innecesarios y utilizando características de seguridad robustas.
- **Educación Continua**: Mantenerse informado sobre las últimas tácticas y herramientas de seguridad es vital para una defensa efectiva contra ataques.

### Reflexión Final

Este reto nos ha enseñado que comprender profundamente las herramientas y técnicas disponibles puede permitirnos proteger eficazmente nuestros dispositivos. Nos alienta a todos a seguir aprendiendo sobre seguridad cibernética y a tomar medidas activas para proteger nuestros datos y dispositivos de amenazas.

¡Gracias por acompañarnos en este reto y esperamos verlos en el próximo desafío de seguridad!
