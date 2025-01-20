---
title: "De Windows a Linux Mint: El salto definitivo"
date: 2025-01-20 19:20:00 +0200
categories: [Linux, Experiencias]
tags: [Linux, Windows, Experiencias, Migraciones, Optimización, Gaming]
image: /img/posts/LinuxMint/LinuxMintPerformance.png
---
# 🌟 Mi Camino: De Windows a Linux Mint

Cambiar de Windows a Linux Mint como mi sistema principal no fue una decisión fácil, pero sí necesaria. Lo que comenzó como un experimento para mejorar el rendimiento en juegos y virtualización terminó convirtiéndose en un cambio definitivo. Aquí comparto cómo lo hice, qué ajustes realicé y cómo logré pasar de **130-144 FPS en Windows** a **245 FPS estables en Linux Mint**.

---

## 🌐 Contexto Inicial

- **Hardware utilizado:**
  - Procesador: AMD Ryzen 7 2700
  - GPU: NVIDIA GeForce RTX 3060
  - Monitor: 3440x1440 @ 180Hz
  - Disco: M.2 de 1TB dedicado a Linux Mint.
- **Problemas en Windows:**
  - Consumo elevado de recursos en segundo plano.
  - Bajones de rendimiento al usar juegos o máquinas virtuales.
  - Actualizaciones forzadas que interrumpían el flujo de trabajo y causaban inestabilidad.
  - Dificultades para mantener los controladores NVIDIA actualizados, con problemas recurrentes tras ciertas actualizaciones.
  - Configuraciones de red predeterminadas ineficientes, lo que afectaba la estabilidad y velocidad de la conexión.
  - Experiencia menos personalizable, llena de bloatware y funciones que ralentizaban el sistema.

---

## 🔧 Ajustes y Configuración

Cuando decidí dar el salto, instalé **Linux Mint 22.1** como sistema base y configuré **KDE Plasma** como entorno de escritorio. KDE me convenció por lo fácil que es adaptarlo a lo que necesito y porque, tras probarlo, noté una mejora real en cómo el sistema respondía a mis tareas diarias.

### 🎮 **Drivers NVIDIA**

Configurar los controladores fue un auténtico dolor de muelas. En mi caso, tuve que:

1. **Desactivar Secure Boot:** Esto permitió que los drivers oficiales de NVIDIA se instalaran sin problemas.
2. **Optimizar con ********`nvidia-settings`********:** Ajusté parámetros como la tasa de refresco, sincronización vertical y rendimiento máximo.

### 🌐 **Frecuencia del Monitor**

Tuve bastantes quebraderos de cabeza intentando configurar los 180Hz en el monitor. Después de varios intentos, logré solucionarlo ajustando los parámetros HDMI y haciendo algunos retoques en los controladores. Al final, conseguí dejarlo estable a 180Hz, pero no sin antes pelearme un buen rato.

### 📊 **Monitorización con Prometheus y Grafana**

Implementé un sistema de monitorización para identificar cuellos de botella y mejorar el uso de recursos. Las métricas más útiles incluyeron:

- **CPU y RAM:** Identificar procesos que consumían demasiados recursos.
- **Carga de GPU:** Ajustar las configuraciones gráficas según el uso.
- **Máquinas virtuales:** Equilibrar recursos para que los juegos y las VMs funcionaran sin interferencias.

Estas herramientas fueron clave para optimizar el rendimiento del sistema y priorizar los procesos más importantes.

---

## 📊 Resultados en FPS

El cambio no solo solucionó los problemas de rendimiento, sino que logró resultados que ni siquiera me imaginaba:

- **En Windows:** FPS promedio entre 130-144, con bajones ocasionales.
- **En Linux Mint:** FPS estables alrededor de 245, incluso en configuraciones gráficas altas.

### 🚀 **Factores Clave de la Mejora**

1. **Eficiencia de recursos:** Linux Mint utiliza menos RAM y CPU en segundo plano.
2. **Controladores optimizados:** Configuración detallada con herramientas específicas.
3. **Monitorización precisa:** Identificar y eliminar procesos innecesarios.

---

## 🎉 Reflexión Final

Cambiar a Linux Mint no fue sencillo, pero al final resultó ser la mejor decisión. Ahora tengo un sistema que no solo funciona mejor, sino que se adapta totalmente a lo que necesito en mi día a día. Aunque hubo momentos en los que me pregunté si valía la pena, ver los resultados ha dejado claro que sí.

Si alguna vez te planteas dar el salto, mi consejo es que te lances. Puede que te lleve algo de tiempo acostumbrarte, pero con un poco de paciencia, descubrirás un sistema que te da más control y, sobre todo, un rendimiento espectacular.

