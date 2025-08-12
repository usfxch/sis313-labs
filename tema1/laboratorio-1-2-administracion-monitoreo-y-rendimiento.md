# Laboratorio 1.2: Administración, Monitoreo y Rendimiento de Servidores Ubuntu Server 24.04 LTS

## 🎯 Objetivo del Laboratorio

- **Instalar y configurar** un entorno de servidor virtualizado con Ubuntu Server.
- **Repasar y aplicar** comandos fundamentales de administración del sistema operativo **GNU/Linux**.
- **Identificar y utilizar** herramientas de monitoreo para evaluar el rendimiento del servidor (CPU, memoria, disco, red).
- **Analizar el impacto** de una carga de trabajo en el rendimiento del servidor.



## 🛠️ Sección 1: Preparación del Entorno Virtualizado

### 1.1 Guía de Instalación de VirtualBox y Ubuntu Server

1.  **Descargar VirtualBox y Ubuntu Server:** Obtén el instalador de VirtualBox y la imagen ISO de **Ubuntu Server 24.04 LTS** desde sus sitios web oficiales.
2.  **Instalar VirtualBox:** Sigue los pasos del instalador de VirtualBox en tu sistema operativo.
3.  **Crear una nueva Máquina Virtual:**
    - Abre VirtualBox y haz clic en **"Nueva"**.
    - Asigna un nombre (ej. `Lab-Ubuntu-Server-24.04`).
    - Selecciona **Linux** y **Ubuntu (64-bit)**.
    - Asigna **2048 MB de RAM** y **2 núcleos de CPU**.
    - Crea un disco duro virtual de al menos **20 GB**.
4.  **Configurar la red:** En la configuración de la VM, selecciona **"Adaptador Puente"** para el adaptador de red.
5.  **Instalar el SO:** Inicia la VM, selecciona la ISO de Ubuntu Server y sigue las instrucciones del instalador.



## 💻 Sección 2: Repaso de Comandos Fundamentales

### 2.1 Comandos de Administración del Sistema y Paquetes

- `sudo`: **Ejecuta un comando con privilegios de superusuario.**
  ```bash
  sudo apt update
  ```
- `apt`: Gestiona paquetes en Ubuntu.
  ```bash
  sudo apt install htop
  ```

- `systemctl`: Controla los servicios del sistema.
  ```bash
  sudo systemctl status ssh

- `useradd` / `passwd`: Crea usuarios y les asigna contraseñas.
  ```bash
  sudo useradd -m nuevo_usuario
  ```
  ```bash
  sudo passwd nuevo_usuario
  ```

### 2.2 Comandos de Gestión de Archivos y Directorios

- `ls`: Lista el contenido de un directorio.
  ```bash
  ls -l
  ```
- `cd`: Cambia el directorio de trabajo.
  ```bash
  cd /var/log
  ```
- `pwd`: Muestra el directorio actual.
  ```bash
  pwd
  ```
- `chmod` / `chown`: Cambia permisos y propiedad de archivos.
  ```bash
  chmod 755 mi_script.sh
  ```
  ```bash
  sudo chown usuario:grupo archivo.txt
  ```
- `cat` / `less` / `tail`: Visualiza el contenido de archivos.
  ```bash
  tail -f /var/log/syslog
  ```

### 2.3 Comandos de Redes

- `ping`: Prueba la conectividad de red.
  ```bash
  ping google.com
  ```

- `ip addr`: Muestra las direcciones de red del sistema.
  ```bash
  ip addr show
  ```

- `netstat` / `ss`: Muestra el estado de las conexiones de red.
  ```bash
  ss -t
  ```

### 2.4 Comandos de Monitoreo y Rendimiento

- `top` / `htop`: Muestran el uso dinámico del CPU, memoria y procesos.
  ```bash
  htop
  ```

- `df`: Informa sobre el uso del espacio en disco.
  ```bash
  df -h
  ```

- `dd`: Simula cargas de escritura en el disco.
  ```bash
  dd if=/dev/zero of=archivo_grande.dat bs=1M count=2048
  ```

- `iostat`: Estadísticas de E/S de los dispositivos de almacenamiento.
  ```bash
  iostat -x
  ```

- `nload`: Monitoriza el tráfico de red en tiempo real.
  ```bash
  nload
  ```

- `kill`: Termina un proceso por su ID (PID).
  ```bash
  kill 1234
  ```

 

## ⚙️ Sección 3: Ejercicio Práctico de Monitoreo y Rendimiento

Tu tarea es simular una carga de trabajo en el servidor y monitorear cómo afecta a sus recursos.

1.  **Instalación de Herramientas de Monitoreo**: Instala `htop`, `sysstat` y `nload` usando `sudo apt install`.

2.  **Línea Base del Rendimiento**: Usa `htop`, `df -h` y `nload` para observar el estado del servidor en reposo y tomar notas.

3.  **Simulación de Carga de Trabajo**:

    - En una terminal, genera una carga alta en la CPU con un bucle:

      ```bash
      while true; do :; done &
      ```

    - En otra terminal, simula una carga de disco con `dd`:

      ```bash
      dd if=/dev/zero of=archivo_grande.dat bs=1M count=2048
      ```

3.  **Monitoreo del Impacto**: Observa los cambios en el rendimiento con `htop`, `iostat -x` y `nload` mientras se ejecutan las tareas. Identifica y termina el proceso de carga con `kill`.

## ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en el siguiente informe detallado que deberás presentar:

1.  **Configuración del Entorno**

    - **Captura de Pantalla**: Muestra una captura de pantalla de la terminal de tu VM con la versión de Ubuntu Server.

    - **Instrucciones**: Describe los pasos clave que seguiste para instalar y configurar la VM.

2. **Comandos y Salidas**

    - **Capturas de Pantalla**: Presenta capturas de pantalla de la ejecución de al menos 5 comandos diferentes de las Secciones 2.1, 2.2 y 2.3, demostrando que has practicado con ellos.

3. **Análisis de Rendimiento**

    - **Línea Base**: Describe el uso de CPU, memoria y disco que observaste con htop y df -h en reposo.

    - **Impacto de la Carga de CPU:**

        - **Comando utilizado**: ¿Qué comando de carga de CPU usaste?

        - **Análisis**: Describe cómo cambió el uso de CPU en `htop` y qué porcentaje de carga observaste.

    - **Impacto de la Carga de Disco:**

        - **Comando utilizado**: ¿Qué comando de carga de disco usaste (`dd`)?

        - **Análisis**: Explica cómo iostat y el tiempo que tomó el comando `dd` reflejan la carga de trabajo en el disco.

    - **Identificación de Procesos:**

        - **Comando**: Muestra el comando htop o top que usaste para identificar el proceso de carga de CPU.

        - **Terminación**: Muestra el comando kill que usaste para finalizar el proceso.

4. **Conclusiones**

    - **Resumen:** Escribe un breve resumen de las lecciones aprendidas sobre cómo el monitoreo de rendimiento es crucial para diagnosticar problemas en un servidor.

    - **Reflexión:** ¿Qué métricas te parecieron más útiles para identificar los cuellos de botella?