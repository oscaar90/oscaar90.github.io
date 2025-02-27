---
title: "Emulación de Android en Linux: Un camino lleno de obstáculos" 
date: 2025-02-27 18:20:00 +0200 
categories: [Linux, Emulación Android] 
tags: [Android, Emulación, Linux, Virtualización, Docker]
image: /img/posts/EmulacionAndroidLinux/android-emulation.png
---

# 📌 El desafío de emular Android en Linux
El verdadero desafío de hoy no ha sido simplemente emular Android en Linux, ya que existen soluciones como Anbox, Waydroid , Genymotion... Que funcionan perfectamente en Linux y Windows.

Lo que realmente quería conseguir era emular Android en una máquina virtual, algo que no pensaba que me daria tantos dolores de cabeza debido a múltiples problemas de compatibilidad y configuración.

Después de múltiples intentos y ajustes, logré hacerlo funcionar con Android-x86 en VirtualBox, modificando el GRUB para solucionar los errores de arranque.

## 🔷 Primer intento: DockerAndroid en un contenedor
Al principio, probé DockerAndroid, una imagen que emula un dispositivo Android dentro de un contenedor Docker con acceso por VNC.

### 🚧 Errores encontrados:
Aunque la emulación cargaba correctamente en http://localhost:6080, adb connect fallaba con "Connection refused".
adb devices dentro del contenedor no mostraba ningún dispositivo en ejecución.
No había comunicación entre el emulador y el sistema anfitrión.
ssds
### ✅ Solución parcial:

Ejecutando: 


```bash docker run -d --privileged --name android-container
-p 6080:6080 -p 5555:5555
-e DEVICE="Samsung Galaxy S10"
-e EMULATOR_ARGS="-no-window -gpu swiftshader_indirect -no-audio"
--shm-size=8g budtmo/docker-android:emulator_13.0 
``` 

Pude acceder al emulador desde el navegador en http://localhost:6080, pero ADB no detectaba el dispositivo, lo que hacía imposible este método para lo que necesitaba.

Decidí intentarlo con Android-x86 en VirtualBox.

## 🔶 Segundo intento: Android-x86 en VirtualBox
Después de instalar Android-x86 en VirtualBox, me encontré con un bucle infinito en la pantalla de inicio.

Investigando, descubrí que el problema estaba en la configuración de GRUB, pero cada vez que lo modificaba en el arranque manualmente, los cambios no se guardaban tras reiniciar.

### 🔥 Solución definitiva: Modificar GRUB y guardar los cambios

Para hacer persistente la configuración del GRUB en Android-x86, realicé los siguientes pasos:

1️⃣ Abrir la consola tras cargar Android
Pulsa Alt + F1 para acceder a la terminal.

2️⃣ Montar el disco virtual de Android-x86
```bash 
mkdir /mnt/sda mount /dev/block/sda1 /mnt/sda 
```

3️⃣ Editar el archivo del menú de GRUB
```bash 
vi /mnt/sda/grub/menu.lst 
```

4️⃣ Modificar la línea del kernel
Busqué la primera entrada de arranque y cambié quiet por: 
```bash 
nomodeset root=/dev/ram0 androidboot.selinux=permissive 
```
5️⃣ Guardar los cambios en vi
Pulsa Esc y escribe :wq, luego presiona Enter para guardar y salir.

6️⃣ Reiniciar el sistema
```bash 
reboot 
```




### 📌 Explicación de los parámetros
En el archivo menu.lst de GRUB, agregamos los siguientes parámetros al kernel:

```plaintext 
androidboot.selinux=permissive nomodeset xforcevesa 
```

Cada uno cumple una función clave para resolver los problemas de arranque y compatibilidad con la máquina virtual.

#### 🔷 1. androidboot.selinux=permissive
SELinux (Security-Enhanced Linux) es un módulo de seguridad en Android y Linux que restringe el acceso de procesos y usuarios al sistema.
Android normalmente se ejecuta con SELinux en modo "enforcing", lo que significa que aplica estrictamente sus reglas de seguridad.
¿Qué hace este parámetro?
Cambia el modo de SELinux a "permissive".
En este modo, SELinux no bloquea acciones potencialmente peligrosas, solo las registra.
Evita problemas con permisos y bloqueos de procesos en el arranque.

#### 📌 ¿Por qué era necesario?

VirtualBox y Android-x86 no están optimizados para funcionar juntos.
SELinux en modo "enforcing" puede bloquear el acceso a controladores de pantalla, entradas de teclado o incluso algunos módulos del kernel, impidiendo el arranque.
Poner SELinux en modo "permissive" hace que el sistema sea más tolerante y no bloquee procesos esenciales.

#### 🔶 2. nomodeset
Este parámetro desactiva el "modo gráfico automático" del kernel de Linux.
Normalmente, el kernel detecta y configura automáticamente los controladores gráficos, pero en VirtualBox esto puede fallar.
¿Qué hace este parámetro?
Evita que el kernel cargue drivers gráficos que puedan ser incompatibles.
Obliga al sistema a usar un controlador de gráficos genérico hasta que el entorno gráfico cargue correctamente.

#### 📌 ¿Por qué era necesario?

Android-x86 puede intentar cargar controladores gráficos no compatibles con VirtualBox, causando pantallas negras o bloqueos en el arranque.
nomodeset evita esto y permite que el sistema se inicie en un modo básico.

## 🎯 Conclusión
Los parámetros evitan que Android-x86 intente usar controladores incompatibles y relajan las restricciones de seguridad para permitir que el sistema se ejecute correctamente en VirtualBox.

