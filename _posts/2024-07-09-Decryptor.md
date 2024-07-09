---
title: "TheHackerLabs - Principiante : [Decryptor]"
date: 2024-07-09 14:35:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/Decryptor/decrypt.jpg
---

## Introducción

Bienvenidos al desafío **"Decryptor"**, un CTF diseñado para principiantes que se lleva a cabo en un sistema operativo basado en Linux. Este reto, creado por `@murrusko`, ofrece una excelente oportunidad para que los principiantes se familiaricen con las operaciones básicas de Linux en un contexto de CTF.

🌐 [**Web oficial del CTF: TheHackersLabs - Decryptor**](https://thehackerslabs.com/decryptor/){:target="_blank"}

## Identificación de la Víctima con Arp-Scan

Iniciamos el reto utilizando `arp-scan` para identificar la dirección IP de la víctima dentro de nuestra red local. Este es un paso crucial para asegurar que los siguientes ataques sean dirigidos correctamente.

Para más detalles sobre cómo realizar este paso, he escrito un post detallado que puedes encontrar aquí:
🌐 [**Consiguiendo la IP de la Víctima en CTFs**](/posts/consiguiendo-la-ip-victima-CTF/){:target="_blank"}

### Ejecución de Arp-Scan

Desde nuestra terminal, lanzamos el siguiente comando:

```bash
arp-scan -I eth0 --localnet
```
![Imagen del resultado de arp-scan](/img/posts/CTF/Decryptor/arpscan.png)

## Primer Escaneo con Nmap

Una vez identificada la dirección IP de la víctima, procedimos con un escaneo exhaustivo de todos los puertos utilizando `nmap`.


### Explicación de los Parámetros Utilizados

- **-p-**: Este parámetro indica que se deben escanear todos los puertos (del 1 al 65535).
- **-vvv**: Aumenta el nivel de verbosidad, proporcionando más detalles sobre el proceso del escaneo.
- **-n**: No resuelve nombres DNS, lo que acelera el escaneo al evitar consultas DNS.
- **-sS**: Realiza un escaneo SYN (semiabierto), que es más rápido y sigiloso que un escaneo completo.
- **-Pn**: Omite el descubrimiento de hosts (no ping), asumiendo que el host está activo.
- **--min-rate 2000**: Establece la tasa mínima de paquetes enviados por segundo a 2000.
- **192.168.1.56**: Especifica la dirección IP del objetivo.
- **-oG allports_Encrypt**: Guarda los resultados en un archivo con formato greppable (fácil de analizar).

### Razón para Usar `--min-rate 2000` en Lugar de `5000`

El parámetro `--min-rate` controla la velocidad a la que `nmap` envía los paquetes durante el escaneo. Establecer una tasa de 2000 paquetes por segundo es un equilibrio entre velocidad y fiabilidad. Usar una tasa más alta, como 5000 paquetes por segundo, podría llevar a:

- **Pérdida de Paquetes**: Las redes y dispositivos podrían descartar paquetes debido a la alta carga, resultando en resultados menos fiables.
- **Detección y Bloqueo**: Los sistemas de detección de intrusiones (IDS) y cortafuegos podrían identificar y bloquear el escaneo debido a la alta tasa de tráfico, dificultando el escaneo exitoso.
- **Saturación de la Red**: En redes más lentas o congestionadas, una tasa alta podría saturar la red, afectando otros servicios y dispositivos.

Por estas razones, utilizamos una tasa de 2000 paquetes por segundo, lo cual proporciona un buen balance entre velocidad y precisión, reduciendo el riesgo de que los paquetes sean descartados o detectados como actividad maliciosa.


```bash
❯ nmap -p- -vvv -n -sS -Pn --min-rate 2000 192.168.1.56 -oG allports_Encrypt
```



![Imagen del primer escaneo nmap](/img/posts/CTF/Decryptor/nmapscan1.png)
## Escaneo Detallado de Puertos Específicos

Tras identificar los puertos abiertos en el primer escaneo, realizamos un escaneo más detallado en los puertos que parecían más interesantes.

### Explicación de los Parámetros Utilizados

- **-sCV**: Realiza un escaneo de detección de versiones y scripts (equivalente a `-sC` para ejecutar scripts de detección y `-sV` para detectar versiones de servicios).
- **-p22,80,2121**: Especifica los puertos a escanear, en este caso, el 22 (SSH), el 80 (HTTP) y el 2121 (FTP alternativo).
- **192.168.1.56**: Especifica la dirección IP del objetivo.
- **-oN ports_Encrypt**: Guarda los resultados en un archivo en formato normal (legible para humanos).

Realizamos el escaneo detallado utilizando el siguiente comando:

```bash
nmap -sCV -p22,80,2121 192.168.1.56 -oN ports_Encrypt
```

Este escaneo nos permitirá obtener más información sobre los servicios que están corriendo en estos puertos, incluyendo sus versiones y posibles vulnerabilidades asociadas.

![Imagen del escaneo detallado nmap](/img/posts/CTF/Decryptor/nmapscan2.png)

### Análisis del Puerto 2121

El puerto `2121` nos llamó especialmente la atención. Aunque no está asignado oficialmente a ningún servicio específico por la IANA, es comúnmente utilizado como una alternativa al puerto estándar `21` para FTP (File Transfer Protocol).

![Imagen relacionada con el puerto 2121](/img/posts/CTF/Decryptor/port2121.png)
















## Investigación de la Página Web

Al investigar el servicio HTTP identificado durante el escaneo de puertos, descubrimos que estaba corriendo un servidor Apache aparentemente estándar. 

![Imagen del código fuente de la página Apache](/img/posts/CTF/Decryptor/apache-source.png)


Decidimos inspeccionar el código fuente de la página para ver si había algo fuera de lo común. Al acceder al código fuente, notamos inmediatamente una anomalía: la barra de desplazamiento sugería que la página era más "larga" de lo que una página por defecto de Apache debería ser.

![Imagen de la barra de desplazamiento en el código fuente](/img/posts/CTF/Decryptor/scrollbar.png)

Continuando con la exploración hasta el final del código, empezamos a obtener nuestras "recompensas".


![Imagen del hallazgo en el código fuente](/img/posts/CTF/Decryptor/found-in-source.png)

### Descubrimiento en el Código: Brainfuck

La cadena que encontramos resultó estar escrita en **Brainfuck**, un lenguaje de programación esotérico conocido por su extrema simplicidad y dificultad de uso práctico.

#### Características de Brainfuck:

- **Modelo de Cinta**: Opera con una cinta de memoria teóricamente infinita dividida en celdas, cada una inicialmente en cero. Un puntero de datos navega por estas celdas.
- **Conjunto de Comandos**: Incluye movimientos del puntero (`>`, `<`), incrementos y decrementos (`+`, `-`), lectura y escritura de bytes (`,`, `.`), y bucles (`[`, `]`).

Cuidado con dejar <!-   -> ya que es html, por lo que la cadena encontrada en el código fuente es la siguiente:

```
++++++++++[>+++++++++++>++++++++++>+++++++++++>+++++++++++>+++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++++>+++++++++++>++++++++++>++++++++++++>++++++++++++>++++++++++>++++++++++<<<<<<<<<<<<<<<-]>-.>---.>++++.>-----.>+.>+.>---.>----.>-----.>--.>+.>----..>---.>-.>+.
```

Esta cadena es un ejemplo típico de cómo Brainfuck puede ser usado para ocultar mensajes o comportamientos dentro de una aplicación de manera que sea difícil de detectar a simple vista.



## Descifrado del Mensaje con Brainfuck

Utilizamos una herramienta en línea para descifrar el mensaje oculto en el código Brainfuck. La herramienta elegida fue la página de dcode, que proporciona un decodificador eficaz para este tipo de lenguajes esotéricos.

🌐 [**dcode - Brainfuck Language Decoder**](https://www.dcode.fr/brainfuck-language)



### Siguiente Pista: marioeatslettuce

El mensaje descifrado nos proporcionó la siguiente pista: `marioeatslettuce`. Este descubrimiento nos lleva a una nueva etapa de la investigación, donde necesitamos aplicar diferentes métodos y herramientas para avanzar.

![Imagen de la herramienta dcode descifrando el mensaje Brainfuck](/img/posts/CTF/Decryptor/dcode.png)

## Pruebas en Curso

En este punto del reto, nos enfrentamos a varios elementos desconocidos y caminos posibles:

- **SSH**: Acceso denegado sin credenciales.
- **vsftpd**: Igualmente inaccesible sin credenciales.
- **marioeatslettuce**: Aparenta ser una contraseña, pero no sabemos dónde utilizarla.

Suponiendo que `marioeatslettuce` podría ser una contraseña, y que está compuesta de un nombre, dedujimos que el login podría ser `mario` y la contraseña `eatslettuce`.

### Intentos de Acceso

Probamos las siguientes combinaciones sin éxito:
- **Login**: mario
- **Password**: eatslettuce

Luego intentamos:
- **Login**: mario
- **Password**: marioeatslettuce

**¡Bingo!** Logramos acceso por el puerto vsftpd 2121.
**Por otra parte** mientras realizabamos el intento de logins `manual`por `logica`, dejamos varios escaneos corriendo. No tenemos que ir a lo loco, pero tampoco ir a paso de tortuga.

Así que de nuevo.... **¡Bingo!** Hydra nos da el mismo resultado

![Imagen de la sesión exitosa en vsftpd](/img/posts/CTF/Decryptor/vsftpd-login.png)

### Reflexión sobre el Proceso de Prueba

Es esencial entender que en un CTF, como en la seguridad informática en general, no hay un único método correcto. Cada participante puede seguir distintos caminos para llegar al mismo resultado. Lo importante es ser flexible y adaptativo, utilizando todas las herramientas y lógica a nuestro alcance para superar los desafíos presentados.

## Finalización de Pruebas

Con la pista descifrada y el acceso conseguido, hemos superado una parte significativa del reto. Ahora nos preparamos para la siguiente fase, donde profundizaremos en el sistema que acabamos de acceder.











## Acceso al Servidor via FTP

Una vez dentro del servidor a través de FTP en el puerto 2121, comenzamos la exploración de los archivos disponibles.

![Imagen de la sesión FTP mostrando archivos disponibles](/img/posts/CTF/Decryptor/ftp-session.png)

### Descubrimiento de un Archivo Keepass

Uno de los hallazgos más interesantes fue un archivo de Keepass, que podría contener información valiosa. Utilizamos el comando `get` para descargarlo a nuestra máquina local.

![Imagen de la descarga del archivo Keepass](/img/posts/CTF/Decryptor/keepass-download.png)

### Exploración de Directorios y Descubrimiento de Flags

Además del archivo Keepass, exploramos otros directorios y encontramos un fichero llamado `user.txt`, que contenía la flag de usuario. Este archivo también fue descargado usando `get`.

![Imagen de la descarga del fichero user.txt](/img/posts/CTF/Decryptor/user-txt-download.png)

## Intento de Crackeo del Archivo Keepass

Con el archivo Keepass en mano, el siguiente paso fue intentar acceder a su contenido, el cual estaba protegido por contraseña.

### Creación de Hash del Fichero Keepass

Para esto, generamos un hash del archivo de Keepass, utilizando herramientas adecuadas para este propósito.

![Imagen del proceso de creación del hash del fichero Keepass](/img/posts/CTF/Decryptor/keepass-hash.png)

### Descubrimiento de la Contraseña

Con el hash en mano, procedimos a usar herramientas de crackeo de contraseñas para descubrir la clave del archivo Keepass. Tras algunas pruebas, logramos obtener la contraseña.

![Imagen del proceso de obtención de la contraseña](/img/posts/CTF/Decryptor/password-found-1.png)
![Imagen de la contraseña obtenida](/img/posts/CTF/Decryptor/password-found-2.png)













## Acceso por SSH

Con la contraseña obtenida del archivo Keepass, finalmente logramos acceso al sistema a través de SSH, el último servicio que nos quedaba por explorar.

![Imagen de la sesión SSH](/img/posts/CTF/Decryptor/ssh-login.png)

### Análisis del Sistema con LinPEAS

Una vez dentro del sistema, decidimos ejecutar LinPEAS, una herramienta de enumeración para mejorar nuestro entendimiento de la configuración y posibles vulnerabilidades del sistema.

![Imagen de la descarga de LinPEAS](/img/posts/CTF/Decryptor/linpeas-download.png)
![Imagen de la ejecución de LinPEAS](/img/posts/CTF/Decryptor/linpeas-run.png)


### Descubrimiento de Vulnerabilidades

LinPEAS reveló dos vulnerabilidades potenciales que podríamos explotar:

- Con nuestro usuario `chiquero`, tenemos la capacidad de ejecutar `/usr/bin/chown` con permisos de administrador.

![Imagen de la vulnerabilidad con chown](/img/posts/CTF/Decryptor/chown-vulnerability.png)

### ¿Edición del Fichero /etc/passwd?

Una de las posibilidades que nos brinda esta vulnerabilidad es la capacidad de editar el fichero `/etc/passwd`, lo que podría permitirnos escalar privilegios o manipular cuentas de usuario.

![Imagen de la edición de /etc/passwd](/img/posts/CTF/Decryptor/etc-passwd.png)

**"El Silencio de los Corderos":** ``"Nunca sabes los ojos que te están mirando."``








## Escalada de Privilegios mediante Edición Directa del `/etc/passwd`

Aprovechando una configuración permisiva en la máquina, descubrimos que teníamos los permisos necesarios para editar directamente el archivo `/etc/passwd`. Este archivo es crucial en sistemas Unix y Linux, ya que almacena información esencial sobre cada cuenta de usuario.

### Generación de un Nuevo Hash de Contraseña

Primero, utilizamos el comando `openssl passwd` para generar un hash de contraseña segura, que luego utilizaremos para modificar el archivo `/etc/passwd` y ganar acceso como superusuario. El comando utilizado fue:

```bash
openssl passwd holaroot
```

Esto nos proporcionó el hash necesario para proceder con la modificación.

### Modificación del Archivo `/etc/passwd`

Con el hash en mano, procedimos a modificar el fichero `/etc/passwd`. Con el nuevo `root` y los usuarios existentes.


![Imagen de la edición de /etc/passwd](/img/posts/CTF/Decryptor/root.png)

![Imagen de la edición de /etc/passwd](/img/posts/CTF/Decryptor/root1.png)

![Imagen de la edición de /etc/passwd](/img/posts/CTF/Decryptor/root3.png)



Con este nivel de acceso, ahora tenemos control total sobre la máquina, permitiéndonos explorar y modificar cualquier aspecto del sistema operativo.

Y hasta aquí ... ya somos root. Las maravillas de no configurar correctamente los permisos, lo que nos puede conllevar....

"Nunca sabes los ojos que te están mirando." — una advertencia siniestra que nos recuerda por qué medidas como estas son esenciales en la seguridad informática. Con este nivel de acceso, ahora tenemos control total sobre la máquina, permitiéndonos explorar y modificar cualquier aspecto del sistema operativo.





## Conclusión

### Herramientas Utilizadas

Durante este writeup, hemos utilizado una variedad de herramientas esenciales para la seguridad informática y el hacking ético:

- `arp-scan`: Para identificar dispositivos en la red local.
- `nmap`: Para escanear puertos y servicios.
- `openssl passwd`: Para generar hashes de contraseñas.
- `LinPEAS`: Para enumeración y descubrimiento de vulnerabilidades.
- Clientes `FTP` y `SSH`: Para conectarnos y explorar los servicios disponibles.

### Medidas de Protección

Para protegernos contra los ataques que hemos explorado en este writeup, podríamos implementar las siguientes medidas:

- **Configuración Correcta de Permisos**: Asegurarse de que los permisos de archivos críticos como `/etc/passwd` estén correctamente configurados.
- **Actualización Regular del Sistema**: Mantener el sistema operativo y las aplicaciones actualizadas para protegerse contra vulnerabilidades conocidas.
- **Seguridad en la Configuración de Servicios**: Configurar servicios como FTP y SSH con medidas de seguridad adecuadas, como la autenticación multifactor y el uso de claves SSH en lugar de contraseñas.
- **Monitoreo y Auditoría**: Implementar herramientas de monitoreo y auditoría para detectar y responder rápidamente a actividades sospechosas.

### Lecciones Aprendidas

De este writeup hemos aprendido varias lecciones importantes:

- La importancia de revisar y asegurar la configuración de permisos en el sistema.
- Cómo utilizar herramientas de enumeración y escaneo para descubrir vulnerabilidades.
- La necesidad de estar siempre alerta y preparado para detectar y mitigar ataques potenciales.

### Cosas a Mejorar

Siempre hay áreas en las que podemos mejorar:

- **Profundización en la Seguridad**: Continuar aprendiendo y mejorando nuestras habilidades en seguridad informática.
- **Mejor Documentación**: Documentar más detalladamente cada paso y comando utilizado durante el writeup para futuras referencias.
- **Exploración de Alternativas**: Investigar y probar diferentes métodos y herramientas para abordar los mismos problemas, aumentando así nuestro repertorio de soluciones.

En resumen, este writeup no solo nos ha ayudado a comprender mejor cómo explotar ciertas vulnerabilidades, sino también a reconocer la importancia de la configuración y seguridad adecuadas en sistemas informáticos. ¡Gracias por acompañarnos en este desafío y esperamos verte en el próximo!