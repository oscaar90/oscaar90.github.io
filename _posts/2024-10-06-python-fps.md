---
title: "Monitorizando el Caos: Mejora tus FPS y Gráficos con Python"
date: 2024-10-06 14:40:00 +0200
categories: [Windows, Configuraciones]
tags: [Windows, NVIDIA, Optimizacin, Python]
image: /img/posts/python-fps.png
---

# Monitorizando el Rendimiento de CPU y GPU con Python para Optimizar Juegos y Revisar el Sistema

En este post quiero compartir cómo he utilizado un **script en Python** para capturar y analizar el rendimiento de la **CPU** y **GPU** durante sesiones de juegos y también para revisar el estado del sistema en reposo. Este enfoque te permite ajustar los parámetros de tu tarjeta gráfica de manera más eficiente y asegurarte de que tu sistema esté funcionando correctamente cuando no está bajo carga.

## ¿Por qué es importante monitorizar el rendimiento?

Monitorear el rendimiento de tu equipo durante juegos o tareas intensivas te ayuda a ajustar la configuración de tus gráficos para obtener el mejor equilibrio entre calidad visual y rendimiento. Si experimentas caídas de FPS, sobrecalentamiento o cuellos de botella en tu CPU o GPU, el análisis de estas métricas te proporcionará una visión clara de cómo se comporta tu equipo.

Además, al monitorear el sistema en reposo, puedes detectar si algún proceso en segundo plano está utilizando más recursos de los esperados y asegurarte de que tu sistema está en un estado saludable cuando no está en uso.

## Requisitos previos

Antes de ejecutar el script, necesitas instalar las bibliotecas **psutil** y **GPUtil**, que nos permitirán obtener información sobre el rendimiento de la **CPU** y **GPU**.

Instala las bibliotecas con pip:

```bash
pip install psutil gputil
```

## El Script en Python para capturar métricas de CPU y GPU

A continuación te dejo el script que captura el **uso de CPU**, **uso de GPU** y **temperatura de la GPU** y guarda los datos en un archivo **CSV** para analizarlos más tarde.

```python
import psutil
import GPUtil
import csv
import time
from datetime import datetime

# Función para obtener el uso de CPU
def get_cpu_usage():
    return psutil.cpu_percent(interval=1)

# Función para obtener el uso de la GPU y su temperatura
def get_gpu_info():
    gpus = GPUtil.getGPUs()
    if gpus:
        gpu = gpus[0]  # Si tienes más de una GPU, selecciona la primera
        gpu_usage = gpu.load * 100  # Uso de GPU en porcentaje
        gpu_temp = gpu.temperature  # Temperatura de GPU en grados Celsius
        return gpu_usage, gpu_temp
    else:
        return None, None

# Archivo CSV donde se guardarán los datos
csv_file = "system_metrics.csv"

# Escribir encabezados en el archivo CSV
with open(csv_file, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["Timestamp", "CPU Usage (%)", "GPU Usage (%)", "GPU Temperature (C)"])

print("Iniciando la monitorización...")

try:
    while True:
        # Obtener datos
        cpu_usage = get_cpu_usage()
        gpu_usage, gpu_temp = get_gpu_info()

        # Si se detecta una GPU, guardar los datos
        if gpu_usage is not None and gpu_temp is not None:
            # Crear una marca de tiempo
            timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

            # Escribir los datos en el archivo CSV
            with open(csv_file, mode='a', newline='') as file:
                writer = csv.writer(file)
                writer.writerow([timestamp, cpu_usage, gpu_usage, gpu_temp])

            # Imprimir los datos en pantalla (opcional)
            print(f"{timestamp} | CPU: {cpu_usage}% | GPU: {gpu_usage:.2f}% | Temp GPU: {gpu_temp}°C")
        else:
            print("No se detecta GPU.")

        # Esperar 5 segundos antes de la siguiente lectura
        time.sleep(5)

except KeyboardInterrupt:
    print("\nMonitorización finalizada.")
```

Este script captura las métricas del sistema en intervalos de 5 segundos y las guarda en un archivo CSV que luego podrás analizar.

## Monitorizando mientras juegas

He utilizado este script para monitorizar el rendimiento de mi sistema mientras jugaba a **Overwatch 2**, lo que me permitió ajustar los parámetros de la tarjeta gráfica de manera más eficiente. Durante la monitorización, observé que el uso de la **GPU** alcanzaba entre **90-100%** durante las partidas, mientras que la **temperatura de la GPU** se mantenía en rangos normales (alrededor de **60°C**).

### Mejorando el rendimiento gráfico y los FPS

Al comenzar a monitorizar mi sistema, me di cuenta de que tenía **problemas de FPS**, ya que no lograba superar los **50-80 FPS** en momentos críticos del juego, lo cual afectaba la fluidez y la experiencia de juego. Tras analizar los datos capturados, vi que mi GPU estaba siendo llevada al límite en configuraciones **Ultra**, lo que provocaba caídas en los FPS. Gracias a los datos obtenidos, pude realizar ajustes precisos en la configuración gráfica, bajando algunos parámetros de **Ultra** a **Alto**. Este cambio no solo mejoró la **calidad gráfica**, sino que también optimizó el uso de la GPU. Ahora consigo mantener **FPS estables entre 130 y 144**, aprovechando al máximo mi monitor de **144 Hz**. La monitorización me permitió mejorar tanto el **rendimiento** como los **gráficos** del juego, alcanzando una experiencia mucho más fluida y sin comprometer la calidad visual.

### Visualización del rendimiento durante el juego

Una vez que tienes los datos capturados, puedes analizarlos utilizando herramientas como **Excel** o **Python** para crear gráficos. Aquí te dejo un ejemplo de cómo visualizar estos datos utilizando **Pandas** y **Matplotlib** en Python:

```python
import pandas as pd
import matplotlib.pyplot as plt

# Cargar el archivo CSV
df = pd.read_csv("system_metrics.csv")

# Convertir la columna 'Timestamp' a formato de fecha
df['Timestamp'] = pd.to_datetime(df['Timestamp'])

# Crear el gráfico
plt.figure(figsize=(10,6))
plt.plot(df['Timestamp'], df['CPU Usage (%)'], label='CPU Usage (%)', color='blue')
plt.plot(df['Timestamp'], df['GPU Usage (%)'], label='GPU Usage (%)', color='green')
plt.plot(df['Timestamp'], df['GPU Temperature (C)'], label='GPU Temperature (C)', color='red')

# Añadir etiquetas y título
plt.xlabel('Tiempo')
plt.ylabel('Porcentaje / Temperatura (°C)')
plt.title('Monitorización de CPU y GPU durante el juego')
plt.legend()
plt.grid(True)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Análisis de rendimiento

El análisis mostró que la GPU estaba trabajando de manera óptima durante el juego, alcanzando su máxima capacidad en los momentos más intensos y manteniendo temperaturas estables. Esto me permitió ajustar los gráficos del juego para mejorar el rendimiento sin sacrificar la calidad visual.

## Monitorización del sistema en reposo

También he utilizado este script para verificar el estado de mi sistema en reposo. Los resultados mostraron que el **uso de la CPU** se mantenía en un rango bajo (entre **2% y 5%**), mientras que la **GPU** tenía un uso ocasional de entre **0% y 5%**, con picos esporádicos cuando algunos procesos del sistema activaban la GPU brevemente.

Esto me permitió confirmar que mi sistema estaba funcionando de manera eficiente en reposo, sin procesos en segundo plano que consumieran recursos excesivos.

### Visualización del sistema en reposo

A continuación te dejo un ejemplo de los gráficos generados para el sistema en reposo:

```python
# Cargar los datos del sistema en reposo
df_reposo = pd.read_csv("system_metrics.csv")
df_reposo['Timestamp'] = pd.to_datetime(df_reposo['Timestamp'])

# Graficar los datos
plt.figure(figsize=(10,6))
plt.plot(df_reposo['Timestamp'], df_reposo['CPU Usage (%)'], label='CPU Usage (%)', color='blue')
plt.plot(df_reposo['Timestamp'], df_reposo['GPU Usage (%)'], label='GPU Usage (%)', color='green')
plt.plot(df_reposo['Timestamp'], df_reposo['GPU Temperature (C)'], label='GPU Temperature (C)', color='red')

plt.xlabel('Tiempo')
plt.ylabel('Porcentaje / Temperatura (°C)')
plt.title('Monitorización del sistema en reposo')
plt.legend()
plt.grid(True)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Análisis del sistema en reposo

El análisis reveló que el sistema estaba en un estado saludable cuando no estaba en uso activo. Los picos ocasionales en el uso de la GPU no eran motivo de preocupación, ya que estaban relacionados con procesos en segundo plano y no afectaban la temperatura ni el rendimiento general.

## Conclusión

Este script es una herramienta útil para monitorizar el rendimiento de tu sistema durante juegos y verificar su estado en reposo. Gracias a los datos capturados, es posible realizar ajustes más informados en la configuración gráfica para optimizar la experiencia de juego y asegurarse de que el sistema esté funcionando correctamente en todo momento.

Si te ha sido útil este post, no dudes en compartirlo y seguir explorando formas de mejorar el rendimiento de tu sistema. ¡Hasta la próxima!
