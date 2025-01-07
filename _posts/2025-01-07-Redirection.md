---
title: "BugBountyLabs - Principiante: [Redirection]"
date: 2025-01-07 10:00:00 +0200
categories: [Bug Bounty, Writeups]
tags: [Bug Bounty, Writeups, BugBountyLabs, Docker, Principiante]
image: /img/posts/BBL/Redirection/redirection.png
---

📛 **Nombre:** Redirect | 📈 **Dificultad:** Principiante | 💻 **SO:** Docker | 👨‍💻 **Creador:** Curiosidades De Hackers & El Pingüino de Mario

---

# 🕵️ Resolución del Bug Bounty:  Redirect

**Redirect** es un laboratorio principiante ofrecido por la plataforma [**BugBountyLabs**](https://bugbountylabs.com/){:target="_blank"}. Este laboratorio permite comprender y explotar una vulnerabilidad clásica de redirección abierta, utilizada comúnmente en ataques de phishing. A través de este writeup, exploraremos los pasos para identificar, analizar y explotar la vulnerabilidad, además de reflexionar sobre las posibles medidas de mitigación.

---

## 🌐 Entorno Inicial

Para comenzar, descargamos y ejecutamos el laboratorio provisto en formato Docker. BugBountyLabs nos facilita un script en Python que automatiza la creación del entorno.

### 📥 Descarga y Ejecución del Laboratorio

1. **Listamos el contenido del directorio:**
```bash
└─$ ls
bugbountylabs_redirection.py
                                                                                                                                                                                                        
└─$ sudo python3 bugbountylabs_redirection.py
[sudo] password for oscar: 

██████╗ ██╗   ██╗ ██████╗     ██████╗  ██████╗ ██╗   ██╗███╗   ██╗████████╗██╗   ██╗    ██╗      █████╗ ██████╗ ███████╗
██╔══██╗██║   ██║██╔════╝     ██╔══██╗██╔═══██╗██║   ██║████╗  ██║╚══██╔══╝╚██╗ ██╔╝    ██║     ██╔══██╗██╔══██╗██╔════╝
██████╔╝██║   ██║██║  ███╗    ██████╔╝██║   ██║██║   ██║██╔██╗ ██║   ██║    ╚████╔╝     ██║     ███████║██████╔╝███████╗
██╔══██╗██║   ██║██║   ██║    ██╔══██╗██║   ██║██║   ██║██║╚██╗██║   ██║     ╚██╔╝      ██║     ██╔══██║██╔══██╗╚════██║
██████╔╝╚██████╔╝╚██████╔╝    ██████╔╝╚██████╔╝╚██████╔╝██║ ╚████║   ██║      ██║       ███████╗██║  ██║██████╔╝███████║
╚═════╝  ╚═════╝  ╚═════╝     ╚═════╝  ╚═════╝  ╚═════╝ ╚═╝  ╚═══╝   ╚═╝      ╚═╝       ╚══════╝╚═╝  ╚═╝╚═════╝ ╚══════╝
        
Fundadores
El Pingüino de Mario
Curiosidades De Hackers

Cofundadores
Zunderrub
CondorHacks

Descargando la máquina redirection, espere por favor...

[########################################] 100%
Descarga completa.
La IP de la máquina es -> 172.17.0.2

Presiona Ctrl+C para detener la máquina
```

---

## 🔍 Enumeración

### 🚪 Análisis Inicial

Una vez desplegado el laboratorio y obtenida la dirección IP (`172.17.0.2`), verificamos la conectividad con la máquina mediante un comando `ping`. Este paso asegura que el laboratorio está accesible desde nuestra máquina anfitriona.

### 📡 Comprobación de Conexión

Ejecutamos el siguiente comando:

```bash
ping -c3 172.17.0.2
```

**Salida del comando:**
```bash
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.244 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.050 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.072 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2036ms
rtt min/avg/max/mdev = 0.050/0.122/0.244/0.086 ms
```

### 🛠️ Observaciones
- **Conexión exitosa:** Los paquetes fueron transmitidos y recibidos correctamente, confirmando la conectividad.
- **Sistema operativo:** El TTL (`time to live`) de los paquetes es 64, lo que sugiere que la máquina objetivo es Linux.

Conectividad establecida, procedemos al siguiente paso para explorar los servicios expuestos en la máquina.

---


---

## 🔍 Enumeración

### 🚪 Escaneo de Puertos

Con la conectividad confirmada, realizamos un escaneo completo de puertos en la máquina objetivo utilizando `nmap`. Esto nos permite identificar servicios activos y obtener información detallada sobre ellos.

Ejecutamos el siguiente comando:

```bash
sudo nmap -sSCV -p- -Pn -n --min-rate 5000 172.17.0.2
```

**Salida del escaneo:**
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-07 10:10 CET
Nmap scan report for 172.17.0.2
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Laboratorio de Open Redirect
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.01 seconds
```

### 🛠️ Observaciones del Escaneo
- **Servicio activo:** El puerto 80 está abierto y sirve contenido a través de un servidor web Apache 2.4.62 (Debian).
- **Título del sitio:** "Laboratorio de Open Redirect", lo que confirma que el objetivo está relacionado con el tema del laboratorio.

---

### 🌐 Análisis HTTP

Con el puerto 80 identificado como activo, inspeccionamos el servicio web utilizando **WhatWeb** para obtener más información sobre las tecnologías subyacentes.

Ejecutamos el siguiente comando:

```bash
whatweb http://172.17.0.2
```

**Salida del comando:**
```bash
http://172.17.0.2 [200 OK] Apache[2.4.62], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.62 (Debian)], IP[172.17.0.2], Title[Laboratorio de Open Redirect]
```

### 🛠️ Observaciones del Análisis
- **Servidor web:** Apache 2.4.62 en Debian Linux.
- **Tecnologías:** HTML5.
- **Estado del servidor:** Todo parece estar correctamente configurado, sin errores evidentes en los encabezados HTTP o el contenido inicial.

Con esta información, estamos listos para explorar más a fondo el sitio web y buscar parámetros o funcionalidades susceptibles a un Open Redirect.

![Web](/img/posts/BBL/Redirection/weblocal.png)
---



---

### 🧪 Laboratorio 1: Redirección Básica

Accedemos al **Laboratorio 1**, donde se nos presenta un botón con la funcionalidad de redirigirnos a otro sitio. Al hacer clic, somos redirigidos a `http://google.com`.

#### 🚩 Observación
La URL del botón es la siguiente:
```bash
http://172.17.0.2/laboratorio1/redirect.php?url=http://google.com
```

#### 🛠️ Explotación
Modificamos el parámetro `url` para redirigir a un sitio de nuestra elección. Cambiamos la URL del enlace a:
```bash
http://172.17.0.2/laboratorio1/redirect.php?url=https://thehackerslabs.com/
```

![Web](/img/posts/BBL/Redirection/lab1-1.png)

**Resultado:**
- La aplicación redirige correctamente a `https://thehackerslabs.com/`.

![Web](/img/posts/BBL/Redirection/lab1-2.png)

#### ✅ Conclusión
El laboratorio demuestra un Open Redirect básico, donde la validación del parámetro `url` no existe, permitiendo redirigir a cualquier sitio arbitrario. Este comportamiento puede ser explotado en ataques de phishing u otras técnicas maliciosas.




Laboratorio 1 completado con éxito.

---

---

### 🧪 Laboratorio 2: Redirección con Validación

Accedemos al **Laboratorio 2**, que presenta un botón similar al del Laboratorio 1. Al hacer clic, somos redirigidos a `https://www.google.com`.

#### 🚩 Observación
La URL del botón es la siguiente:
```bash
http://172.17.0.2/laboratorio2/redirect.php?url=https://www.google.com
```

Intentamos modificar el parámetro `url` para redirigir a un sitio diferente:
```bash
http://172.17.0.2/laboratorio2/redirect.php?url=https://elrincondelhacker.es/
```

**Resultado:**
- La aplicación detecta el cambio y muestra un mensaje de alerta:
  > "Redirección no permitida. Solo puedes redirigirte a Google."

---

#### 🛠️ Explotación

Para evadir la validación, utilizamos un método conocido como **redirección con autenticación embebida**. Modificamos la URL para incluir el dominio permitido como parte del usuario en la autenticación HTTP básica:

```bash
http://172.17.0.2/laboratorio2/redirect.php?url=https://www.google.com@elrincondelhacker.es/
```

**Resultado:**
- Al acceder, el navegador muestra un aviso:
  > "You are about to log in to the site 'elrincondelhacker.es' with the username 'www.google.com', but the website does not require authentication. This may be an attempt to trick you. Is 'elrincondelhacker.es' the site you want to visit?"

- Confirmamos con "Yes" y somos redirigidos correctamente a `https://elrincondelhacker.es/`.

![Web](/img/posts/BBL/Redirection/lab2-1.png)

---

#### ✅ Conclusión

La aplicación no valida correctamente la URL al interpretar `https://www.google.com@elrincondelhacker.es/`. Este método permite evadir restricciones al incluir el dominio permitido como un identificador de autenticación en la URL.

Laboratorio 2 completado con éxito.

---


---

### 🧪 Laboratorio 3: Redirección con Validación Estricta

El **Laboratorio 3** tiene un diseño similar a los anteriores, con un botón que redirige a `https://www.google.com`. Sin embargo, la validación en este caso es más estricta, bloqueando los métodos utilizados en los laboratorios previos.

#### 🚩 Observación
El enlace del botón es el siguiente:
```bash
http://172.17.0.2/laboratorio3/redirect.php?url=https://www.google.com
```

Intentamos modificar el parámetro `url` como en los laboratorios anteriores:

1. **Cambio directo de URL:**
   ```bash
   http://172.17.0.2/laboratorio3/redirect.php?url=https://bugbountylabs.com/
   ```
   **Resultado:**  
   > "Redirección no permitida. Solo puedes redirigirte a Google."

2. **Uso del método de autenticación embebida:**
   ```bash
   http://172.17.0.2/laboratorio3/redirect.php?url=https://www.google.com@https://bugbountylabs.com/
   ```
   **Resultado:**  
   > "Redirección no permitida. Solo puedes redirigirte a Google."

A pesar de los intentos, los métodos anteriores fallan debido a las validaciones adicionales implementadas en la aplicación. Sin embargo, al hacer clic en el botón, somos redirigidos correctamente a `https://www.google.com`.

---

#### 🛠️ Explotación

Nos preguntamos: **¿Qué ocurre si usamos un enlace válido de Google pero que apunte a otra página o dominio?**

Modificamos el parámetro `url` para apuntar a una búsqueda de Google, como la siguiente:

```bash
http://172.17.0.2/laboratorio3/redirect.php?url=https://www.google.com/search?q=bugbountylabs
```

**Resultado:**
- La aplicación redirige correctamente al enlace de búsqueda de Google:
  > `https://www.google.com/search?q=bugbountylabs`.

---

#### ✅ Explicación del Método

La validación de la aplicación solo verifica si el dominio base es `www.google.com`, pero no inspecciona el resto del enlace. Esto permite aprovechar cualquier subpágina de Google para redirigir al usuario a un contenido controlado de manera indirecta.

Por ejemplo, en este caso, al usar el enlace de búsqueda `172.17.0.2/laboratorio3/redirect.php?url=https://www.google.com/search?q=bugbountylabs&sca_esv=7caec7b3ee93c0f3&source=hp&ei=JAF9Z9_cKbvj5NoPkObqmA8&iflsig=AL9hbdgAAAAAZ30PNPh_qj6ISjaNs5tFzfOEUQWE5nkJ&ved=0ahUKEwjf_sfVsuOKAxW7MVkFHRCzGvMQ4dUDCBA&uact=5&oq=bugbountylabs`, el usuario es llevado a una página controlada por Google que muestra información sobre el dominio "bugbountylabs.com". Esto puede ser aprovechado para confundir al usuario y hacerlo creer que está accediendo a un sitio confiable.

![Web](/img/posts/BBL/Redirection/lab3-1.png)
![Web](/img/posts/BBL/Redirection/lab3-2.png)


---

#### 🎉 Conclusión del Laboratorio 3

Este laboratorio demuestra cómo incluso con validaciones más estrictas, los Open Redirects pueden ser explotados utilizando enlaces válidos dentro de dominios permitidos. Este método es especialmente útil cuando la aplicación solo permite un conjunto restringido de dominios y valida únicamente el dominio base.

Laboratorio 3 completado con éxito.




---

---

## 🎉 Agradecimientos

Un agradecimiento especial a **Curiosidades De Hackers** y **El Pingüino de Mario** por diseñar este interesante reto en [BugBountyLabs](https://bugbountylabs.com). La creatividad y atención al detalle en cada uno de los laboratorios son un excelente ejemplo de cómo aprender sobre vulnerabilidades en un entorno seguro y controlado.

---

## 🏁 Reflexión Final

Este conjunto de laboratorios nos permitió explorar diferentes enfoques para explotar vulnerabilidades de **Open Redirect**, desde redirecciones básicas hasta métodos más avanzados que implican evadir validaciones. Entre los aprendizajes clave están:

1. **Conocer las validaciones de la aplicación:** Identificar cómo funcionan las restricciones y buscar formas de evadirlas.
2. **Aprovechar comportamientos permisivos:** Como el uso de subdominios o enlaces válidos dentro de un dominio permitido.
3. **Pensar de forma creativa:** Las vulnerabilidades no siempre son evidentes; a veces requieren explorar caminos indirectos para su explotación.

Espero que este writeup haya sido útil para comprender mejor las vulnerabilidades de Open Redirect y cómo abordarlas en un entorno de Bug Bounty. ¡Nos vemos en el próximo desafío! 🚀

---
