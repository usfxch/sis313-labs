# Laboratorio 1.2: Administración, Monitoreo y Rendimiento de Servidores Ubuntu Server 24.04 LTS

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

- **Instalar y configurar** un entorno de servidor virtualizado con Ubuntu Server.
- **Repasar y aplicar** comandos fundamentales de administración del sistema operativo **GNU/Linux**.
- **Identificar y utilizar** herramientas de monitoreo para evaluar el rendimiento del servidor (CPU, memoria, disco, red).
- **Analizar el impacto** de una carga de trabajo en el rendimiento del servidor.

## 🛠️ Sección 1: Preparación del Entorno Virtualizado

### 1.1 Guía de Instalación de VirtualBox y Ubuntu Server

1.  **Descargar VirtualBox y Ubuntu Server:** Obtén el instalador de VirtualBox y la imagen ISO de **Ubuntu Server 24.04 LTS** desde sus sitios web oficiales.
    - [Descarga la última versión de Ubuntu Server](https://ubuntu.com/download/server).
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

Puedes consultar también el documento
[Ubuntu CLI cheat sheet](ubuntu-server-cli-cheat-sheet-02.07.2024-update.pdf), donde tienes un resumen muy completo del uso de comandos para la terminal.

### 2.1 Comandos de Administración del Sistema y Paquetes

- `sudo`: Ejecuta un comando con privilegios de superusuario.

  Utiliza sudo por delante para ejecutar cualquier comando del sistema.
  ```bash
  sudo apt update
  ```

- `apt`: Gestiona paquetes en Ubuntu.

  Actualiza la lista de repositorios de paquetes (software):
  ```bash
  sudo apt update
  ```

  Actualiza los paquetes a sus nuevas versiones:
  ```bash
  sudo apt upgrade
  ```

  Instala un paquete:
  ```bash
  sudo apt install nginx
  ```

  Elimina un paquete:
  ```bash
  sudo apt remove nginx
  ```

  Elimina y borra los archivos de configuración de un paquete:
  ```bash
  sudo apt purge nginx
  ```

- `systemctl`: Controla los servicios del sistema.
  
  Ver estado del servicio:
  ```bash
  sudo systemctl status ssh
  ```
  
  Detener el servicio:
  ```bash
  sudo systemctl stop nginx
  ```

  Iniciar el servicio:
  ```bash
  sudo systemctl start nginx
  ```

  Reiniciar el servicio:
  ```bash
  sudo systemctl start nginx
  ```

  Recarga la configuración de un servicio sin necesidad de reiniciarlo:
  ```bash
  sudo systemctl reload nginx
  ```

- `groupadd`: Crea un grupo de usuarios con el identificador `2000`.

  ```bash
  sudo groupadd -g 2000 nuevo_grupo
  ```

- `useradd` / `usermod` / `userdel`: Crea, modifica y elimina usuarios.

  Crea un usuario con identificador `2001`, para el grupo `2000`, con grupo adicional `sudo`, con una descripción de su nombre `Nuevo Usuario`, creando y asignándole una carpeta en `/home/nuevo_usuario` y con la terminal del tipo `bash`.
  ```bash
  sudo useradd -u 2001 -g 1000 -G sudo -c "Nuevo Usuario" -md /home/nuevo_usuario -s /bin/bash nuevo_usuario
  ```

  Modificar la terminal del usuario:
  ```bash
  sudo usermod -s /bin/sh nuevo_usuario
  ```

  Elimina un usuario y su directorio home:
  ```bash
  sudo userdel -r nuevo_usuario
  ```

- `passwd` / `chage`: Gestiona contraseñas y la política de caducidad.

  Asigna una contraseña al usuario:
  ```bash
  sudo passwd nuevo_usuario
  ```

  Bloquea la cuenta de un usuario:
  ```bash
  sudo passwd -l usuario
  ```

  Desbloquea la cuenta de un usuario:
  ```bash
  sudo passwd -u usuario
  ```

  Muestra la información de caducidad de la contraseña:
  ```bash
  sudo chage -l nuevo_usuario
  ```

### 2.2 Comandos de Gestión de Archivos y Directorios

- `touch`: Crea archivos vacíos o actualiza la marca de tiempo.
  ```bash
  touch nuevo_archivo.txt
  ```

- `ls`: Lista el contenido de un directorio.

  Muestra todos los archivos:
  ```bash
  ls -l
  ```

  Muestra todos los archivos, incluyendo los ocultos:
  ```bash
  ls -la
  ```

- `cd`: Cambia el directorio de trabajo.
  ```bash
  cd /var/log
  ```

- `pwd`: Muestra el directorio actual.
  ```bash
  pwd
  ```

- `mkdir`: Crea un nuevo directorio.

  Crea un directorio:
  ```bash
  mkdir nueva_directorio
  ```

  Crea un directorio y su subdirectorio:
  ```bash
  mkdir -p nuevo_directorio/subdirectorio
  ```

- `cp` / `mv` / `rm`: Copia, mueve o elimina archivos y directorios.

  Copia un directorio recursivamente:
  ```bash
  cp -r nuevo_directorio copia_nuevo_directorio
  ```

  Mueve un archivo a otro directorio:
  ```bash
  mv archivo.txt /tmp/
  ```

  Borra el directorio recursivamente:
  ```bash
  rm -r mi_directorio
  ```

- `find`: Busca archivos en una jerarquía de directorios.

  Busca un archivo por nombre en todo el sistema
  ```bash
  find / -name "archivo.txt"
  ```

- `grep`: Busca patrones de texto dentro de archivos.

  Busca la palabra "error" en un archivo de log:
  ```bash
  grep "error" /var/log/syslog
  ```

- `head` / `tail`: Muestran las primeras o últimas líneas de un archivo.

  Muestra las primeras 5 líneas:
  ```bash
  head -n 5 /etc/passwd
  ```

  Muestra las últimas 10 líneas:
  ```bash
  tail -n 10 /var/log/syslog
  ```

- `chmod` / `chown`: Cambia permisos y propiedad de archivos.

  Cambia los permisos de acceso a un archivo:
  ```bash
  chmod 755 mi_script.sh
  ```

  Cambia la propiedad (usuario y grupo) a un archivo:
  ```bash
  sudo chown usuario:grupo archivo.txt
  ```

- `cat` / `less`: Visualiza el contenido de archivos.
  ```bash
  cat /var/log/syslog
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

- `uname` / `hostnamectl`: Muestran información del sistema.

  Muestra información detallada del kernel y el sistema:
  ```bash
  uname -a
  ```

  Muestra la configuración del `hostname`:
  ```bash
  hostnamectl
  ```

- `lscpu`: Muestra información detallada del CPU.
  ```bash
  lscpu
  ```

- `top` / `htop`: Muestran el uso dinámico del CPU, memoria y procesos.
  ```bash
  htop
  ```

- `free`: Informa sobre el uso de la memoria del sistema.

  Muestra la memoria en megabytes:
  ```bash
  free -m
  ```

- `df`: Informa sobre el uso del espacio en disco.
  ```bash
  df -h
  ```

- `crontab`: Programa tareas para que se ejecuten automáticamente.

  Edita el `crontab` del usuario actual:
  ```bash
  crontab -e
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

- `ps`: Muestra instantanea de procesos en ejecución.
  ```bash
  ps -aux
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

    - **Línea Base**: Describe el uso de CPU, memoria y disco que observaste con `htop` y `df -h` en reposo.

    - **Impacto de la Carga de CPU:**

        - **Comando utilizado**: ¿Qué comando de carga de CPU usaste?

        - **Análisis**: Describe cómo cambió el uso de CPU en `htop` y qué porcentaje de carga observaste.

    - **Impacto de la Carga de Disco:**

        - **Comando utilizado**: ¿Qué comando de carga de disco usaste (`dd`)?

        - **Análisis**: Explica cómo iostat y el tiempo que tomó el comando `dd` reflejan la carga de trabajo en el disco.

    - **Identificación de Procesos:**

        - **Comando**: Muestra el comando `htop`/`top` o `ps` que usaste para identificar el proceso de carga de CPU.

        - **Terminación**: Muestra el comando `kill` que usaste para finalizar el proceso.

4. **Conclusiones**

    - **Resumen:** Escribe un breve resumen de las lecciones aprendidas sobre cómo el monitoreo de rendimiento es crucial para diagnosticar problemas en un servidor.

    - **Reflexión:** ¿Qué métricas te parecieron más útiles para identificar los cuellos de botella?