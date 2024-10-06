---
title: "Problema: Latencia alta en el sistema y fallos en el ratón (clic izquierdo) en Shooters"
date: 2024-09-25 15:17:00 +0200
categories: [Windows, Configuraciones]
tags: [Windows, NVIDIA, Optimizacin, Drivers]
image: /img/posts/Systemlatency.png
---


# Problema: Latencia alta en el sistema y fallos en el ratón (clic izquierdo) en Shooters

## Descripción del problema:
Experimentaba problemas con el clic izquierdo del ratón al jugar a **Shooters**. Tenía que pulsar varias veces el clic izquierdo para que respondiera correctamente, lo cual afectaba negativamente el rendimiento en el juego. Sin embargo, en tareas de ofimática, el ratón funcionaba sin problemas. Esto sugería que el problema estaba relacionado con cómo el sistema manejaba las entradas del ratón bajo condiciones de alta demanda.

### Primer paso: Diagnóstico con **LatencyMon**
Utilizamos **LatencyMon** para analizar la latencia del sistema y verificar si el PC era capaz de manejar tareas en tiempo real, como la respuesta del ratón en juegos.

#### ¿Qué es **LatencyMon**?
**LatencyMon** es una herramienta que mide la capacidad de un sistema para ejecutar tareas en tiempo real sin interrupciones. Monitorea el tiempo que tarda el sistema en procesar interrupciones de hardware y el tiempo que los controladores de dispositivo (como los de red, sonido y almacenamiento) tardan en responder. Si uno de estos controladores tarda demasiado en ejecutar su tarea, esto puede causar problemas de rendimiento, como clics del ratón no registrados o retrasos en el audio.

En la primera prueba de **LatencyMon**, se mostró el siguiente mensaje:
- _"Your system seems to be having difficulty handling real-time audio and other tasks. One problem may be related to power management, disable CPU throttling settings in Control Panel and BIOS setup."_

Esto indicó que había problemas con la latencia relacionados principalmente con la **gestión de energía del procesador** (CPU throttling) y los adaptadores de red.

---

## Paso 2: Optimización de la configuración de energía

### ¿Qué es el **CPU throttling**?
El **CPU throttling** es una técnica utilizada por el sistema operativo y el hardware para reducir la velocidad del procesador (CPU) en momentos en los que no se necesita un alto rendimiento. Esto se hace para reducir el consumo de energía y evitar el sobrecalentamiento. Sin embargo, en situaciones de alta demanda (como los juegos), el **CPU throttling** puede reducir el rendimiento del sistema, generando latencias y retrasos que afectan negativamente la experiencia de usuario, especialmente en tareas en tiempo real como el procesamiento de clics del ratón.

#### Ajustes en el plan de energía de Windows
Para evitar que el procesador redujera su velocidad durante las sesiones de juego, realizamos ajustes en el plan de energía de Windows. Configuramos los **estados del procesador** de la siguiente manera:

1. **Estado mínimo del procesador** al **100%**: Esto garantiza que el procesador siempre opere a su velocidad máxima, incluso en momentos de baja carga.
2. **Estado máximo del procesador** al **100%**: Esto permite que el procesador alcance su máxima capacidad cuando sea necesario, asegurando el mejor rendimiento posible en tareas exigentes como los juegos.

#### Ruta para configurar los estados del procesador:
- **Panel de control** > **Opciones de energía** > **Cambiar la configuración del plan** > **Cambiar la configuración avanzada de energía** > **Administración de energía del procesador**.

El objetivo fue deshabilitar el **CPU throttling** asegurando que el procesador funcione siempre a plena capacidad.

---

## Paso 3: Revisión y ajuste del **BIOS**

### Revisión de la configuración del BIOS
Además de ajustar las opciones de energía en Windows, revisamos la configuración del **BIOS**. El **BIOS** es el firmware que inicializa el hardware cuando arrancas el PC y controla configuraciones críticas, incluyendo cómo se gestiona la energía del CPU.

#### Desactivación de funciones de ahorro de energía en el BIOS
Se desactivaron funciones que podrían estar afectando el rendimiento del procesador, como:
- **Intel SpeedStep**: Reduce dinámicamente la velocidad del CPU para ahorrar energía.
- **C-States**: Ponen el procesador en estados de inactividad profunda para ahorrar energía, pero pueden introducir latencia.
- **AMD Cool’n’Quiet** (para CPUs AMD): Función similar a Intel SpeedStep que reduce la velocidad del procesador para reducir el consumo energético y el calor.

Estos ajustes garantizan que el procesador no entre en modos de bajo consumo que puedan afectar la latencia.

---

## Paso 4: Gestión de los adaptadores de red

### Revisión de adaptadores de red en el **Administrador de dispositivos**
Notamos que en el sistema había numerosos adaptadores de red virtuales instalados debido al uso de software como **VMware**, **VirtualBox** y **NordVPN**. Cada uno de estos adaptadores de red genera interrupciones en el sistema, lo que puede aumentar la latencia y causar problemas en tareas en tiempo real.

#### Deshabilitación de adaptadores de red no utilizados
Para reducir la carga en el sistema, deshabilitamos los adaptadores de red que no se estaban utilizando activamente:
- **Adaptadores de VMware y VirtualBox**: Se deshabilitaron, ya que no eran necesarios en ese momento.
- **Adaptadores VPN no utilizados**: Dado que solo usábamos **NordVPN**, se deshabilitaron otros adaptadores relacionados con diferentes VPNs.

#### Reinstalación y actualización de **NordVPN**
Reinstalamos **NordVPN** para asegurarnos de que su configuración de red estuviera funcionando correctamente, y actualizamos el software para evitar posibles conflictos con otros adaptadores.

---

## Paso 5: Controladores (Drivers)

### Drivers de NVIDIA (nvlddmkm.sys)
El controlador de NVIDIA **nvlddmkm.sys** fue identificado como uno de los principales responsables de la alta latencia, con el siguiente análisis de **LatencyMon**:
   - **DPC Count**: 3,945,175
   - **Highest Execution Time**: 1.096 ms
   - **Total Execution Time**: 64,085.926 ms

Esto sugiere que el controlador de modo kernel de **NVIDIA** está generando latencias importantes, probablemente debido a un conflicto o un controlador desactualizado.

#### Acciones recomendadas:
1. **Actualizar los drivers gráficos**: 
   - Abrir **NVIDIA GeForce Experience** o el software de controladores de NVIDIA y actualizar a la última versión disponible. Esto puede resolver problemas de latencia relacionados con los gráficos.

2. **Reinstalar los drivers utilizando DDU (Display Driver Uninstaller)**:
   - Si ya tienes la última versión de los controladores o si la actualización no soluciona el problema, desinstala completamente los drivers gráficos utilizando **DDU (Display Driver Uninstaller)**.
   - **Descarga DDU** desde [Guru3D](https://www.guru3d.com/files-details/display-driver-uninstaller-download.html).
   - Después de desinstalar, reinstala los controladores gráficos más recientes desde la página oficial de **NVIDIA**.

#### Actualización automática de NVIDIA:
Al abrir **NVIDIA GeForce Experience**, el software inició una actualización automática de los drivers, lo cual puede haber solucionado el problema.

---

## Paso 6: Resultados de la segunda prueba con **LatencyMon**

Después de realizar los ajustes mencionados, ejecutamos nuevamente **LatencyMon** para evaluar la mejora en el rendimiento. Esta vez, el resultado fue positivo:

- _"Your system appears to be suitable for handling real-time audio and other tasks without dropouts."_

Los tiempos de ejecución de los controladores mejoraron significativamente, y los problemas de latencia relacionados con las entradas del ratón desaparecieron.

#### Tiempos de ejecución destacados en **LatencyMon**:
- **nvlddmkm.sys** (NVIDIA): DPC Count: 3,945,175, Highest Execution: 1.096 ms
- **HDAudBus.sys** (Controlador de audio de alta definición): DPC Count: 202,081, Highest Execution Time: 0.217 ms
- **tcpip.sys** (Controlador TCP/IP): DPC Count: 2,872, Highest Execution Time: 0.687 ms

---

## Conclusión:
Gracias a la desactivación de los adaptadores de red no utilizados, la actualización de los drivers de **NVIDIA** y las optimizaciones en la gestión de energía, el sistema ahora es capaz de manejar tareas en tiempo real sin problemas de latencia. Los clics del ratón en **Shooters** responden correctamente, y la experiencia de juego ha mejorado significativamente.

### Recomendaciones:
1. **Mantener deshabilitados los adaptadores de red no utilizados** para evitar conflictos innecesarios.
2. **Actualizar periódicamente los drivers de NVIDIA** y otros controladores importantes (red, audio) para evitar futuros problemas de latencia.
3. **Monitorear el sistema** con **LatencyMon** en caso de que los problemas reaparezcan.

