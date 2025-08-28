# Laboratorio 2.1: Implementación de un servidor virtual con RAID

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

- **Implementar y configurar** un arreglo RAID en un servidor virtual usando discos virtuales.

- **Comprender y demostrar** los beneficios del arreglo RAID en términos de redundancia y rendimiento.

- **Simular y verificar** la tolerancia a fallos al probar la falla de un disco dentro del arreglo.

## 🛠️ Sección 1: Preparación del Entorno Virtual

**Para este laboratorio necesitaremos:**

- **VirtualBox instalado en tu computadora.**
- **Una Máquina Virtual (VM) con:**
    - **Ubuntu Server** en su versión **24.04 LTS** instalado por defecto en un disco virtual.
        - Instalar servidor **OpenSSH**.
    - Acceso a la línea de comandos o terminal (instalar [Warp desde aquí](https://app.warp.dev/referral/3DY6RJ)). 
    - Adaptador de red conectado como **Adaptador puente**.
    - Al menos **dos discos virtuales** para **RAID 1**.
    - Al menos **tres discos virtuales** para **RAID 5**.
    - Al menos **cuatro discos virtuales** para **RAID 10**.
    - Asegúrate de que los discos adicionales tengan un tamaño similar.

## 💻 Sección 2: Práctica guiada

### Creación del Arreglo RAID 1

1. **Creación del Sistema de Archivos y Montaje**
    - **Paso 1.** Antes de crear el RAID, identifica los nombres de los discos adicionales:

        ```bash
        sudo fdisk -l
        ```

    - **Paso 2.** Ahora para crear un arreglo **RAID 1 (Mirroring)** con `/dev/sdb` y `/dev/sdc`:
        ```bash
        sudo mdadm --create --verbose /dev/md1 --level=mirror --raid-devices=2 /dev/sdb /dev/sdc
        ```

         ```bash
        cat /proc/mdstat
        ```
        > Muestra el estado actual de las arreglos RAID de software.

    - **Paso 3.** Crea un sistema de archivos de tipo `EXT4` en el dispositivo `/dev/md1`:
        ```bash
        sudo mkfs.ext4 /dev/md1
        ```

    - **Paso 4.** Crea un directorio llamado `raid1` dentro de `/mnt/` y luego monta el dispositivo RAID 1 `/dev/md1` en ese nuevo directorio:
        ```bash
        sudo mkdir /mnt/raid1
        ```

        ```bash
        sudo mount /dev/md1 /mnt/raid1
        ```

    - **Paso 5.** Escribe algunos archivos de prueba en `/mnt/raid`.

        ```bash
        # Reemplaza con el nombre de usuario y grupo de tu SO
        sudo chown <usuario>:<grupo> /mnt/raid1
        ```

        ```bash
        echo "Hola Mundo" > /mnt/raid1/archivo_de_prueba.txt
        ```
        > Crea un archivo de texto en el directorio del RAID.

        ```bash
        cd /mnt/raid1 && dd if=/dev/zero of=archivo_grande_1.dat bs=1M count=1024
        ```
        > Crea un archivo de 1GB dentro del directorio del RAID.

2. **Configurar el RAID para que inicie desde el arranque**
    - Añadir al archivo `/etc/fstab` el montaje del disco RAID (utiliza el editor `nano`):

        ```bash
        sudo blkid
        ```
        > Muestra información detallada de los dispositivos de bloque, como discos duros y particiones, incluyendo sus etiquetas, tipos de sistema de archivos y, sobre todo, sus `UUID` (Universal Unique Identifiers), los cuales son identificadores únicos universales. Identifica el `UUID` del RAID `md1` y reemplazalo en el siguiente comando.

        ```bash
        UUID=<UUID del RAID>  /mnt/raid1  ext4  defaults,nofail  0  2
        ```
    - Revisa los archivos de configuración del Sistema y de las unidades de disco y reinicia la VM:
        ```bash
        sudo systemctl daemon-reload
        ```

        ```bash
        sudo mount -a
        ```

        ```bash
        sudo reboot
        ```

3. **Simulación de Fallo de Disco**

    - Simularemos la falla de un disco (ejemplo: `/dev/sdb`) marcándolo como defectuoso:

        ```bash
        sudo mdadm --manage /dev/md1 --fail /dev/sdb
        ```
        > Marca el disco `/dev/sdb `como fallido dentro del RAID `/dev/md1`.

        ```bash
        sudo mdadm --manage /dev/md1 --remove /dev/sdb
        ```
        > Elimina el dispositivo `/dev/sdb` del RAID `/dev/md1`.

4. **Verificación de la Integridad de los Datos**

    - Intenta acceder a los archivos que creaste en `/mnt/raid`:

        ```bash
        cd /mnt/raid
        ```

        ```bash
        ls -l
        ```

        ```bash
        cat archivo_de_prueba.txt 
        ```
        > ¿Los datos siguen accesibles?

5. **Estado del Arreglo RAID**

    - Verifica el estado del arreglo RAID con:

        ```bash
        sudo mdadm --detail /dev/md1
        ```
        > Observa el estado del arreglo. ¿Indica que está degradado?
    
6. **Simulación de Reconstrucción (Pasos)**

    Para simular la reconstrucción:

    1. Apaga la VM.
    2. Añade un nuevo disco virtual del mismo tamaño a la configuración de la VM.
    3. Inicia la VM.
    4. Identifica el nuevo disco (ej: `/dev/sdd`).
    5. Añade el nuevo disco al arreglo para iniciar la reconstrucción:
        ```bash
        sudo mdadm --manage /dev/md1 --add /dev/sdd
        ```
    6. Monitorea el progreso de la reconstrucción:
        ```bash
        sudo mdadm --detail /dev/md1
        ```

### Puntos Clave
**En esta práctica hemos observado:**

- Cómo configurar un arreglo RAID por software en un entorno virtualizado.
- La capacidad de RAID 1 para mantener la disponibilidad de los datos ante la falla simulada de un disco.
- El concepto de degradación del arreglo RAID cuando falla un disco.
- El proceso de reconstrucción para restaurar la redundancia.

**La redundancia es crucial para la tolerancia a fallos en sistemas de almacenamiento críticos.**

## ⚙️ Sección 3: Práctica Individual. Implementación de un servidor virtual con RAID 5 y RAID 10

### 📋 Escenario del Laboratorio

La empresa **"TechSolutions Inc."** te ha solicitado que prepares un servidor para una base de datos crítica. Para garantizar la alta disponibilidad y el rendimiento, la directiva ha especificado que el servidor debe tener un sistema de almacenamiento configurado con un arreglo RAID 5 (RAID 10 para el Grupo 2 de Laboratorio). Deberás utilizar un entorno virtualizado para demostrar que el diseño cumple con los requisitos.

### ⚙️ Tareas a Realizar

1. **Preparación del Entorno Virtualizado**

    - Crea una nueva máquina virtual en VirtualBox, asignándole un mínimo de 2 GB de RAM, 2 CPU y un disco de 10 GB.

    - Adiciona 3 o más discos (4 para G2) duros virtuales a la máquina, cada uno de 2 GB de tamaño. Asegúrate de que todos los discos no hayan sido utilizados antes.

    - Instala Ubuntu Server 24.04 LTS en el primer disco (el que se usará para el sistema operativo). La instalación en un disco separado facilitará la práctica.

2. **Práctica Individual:**

    Realiza los siguientes pasos de forma individual:
    1. **Crea un arreglo RAID 5 (RAID 10 para G2)** utilizando `mdadm` con los discos virtuales necesarios en tu máquina virtual, establece el nivel correcto y asigna el nombre de dispositivo `/dev/md5` (`/dev/md10` para G2) al arreglo.
    2. **Crea un sistema de archivos** (`ext4`) en el dispositivo RAID virtual `/dev/md5` (`/dev/md10` para G2).
    3. **Crea un punto de montaje** (ej: `/mnt/raid5` para G2 y `/mnt/raid10` para G2) y monta el sistema de archivos RAID en este punto.
    4. **Crea varios archivos importantes** en el directorio montado
    (`/mnt/raid5` o `/mnt/raid10`) y **copia archivos gran tamaño** (superiores a 100 MB) desde tu host anfitrión a tu máquina virtual utilizando `scp`.
    5. **Simula la falla de uno de los discos virtuales** del arreglo RAID 5 (RAID 10 para G2) y reemplaza con un nuevo disco.
    6. **Verifica que los archivos creados en el paso "iv" sigan accesibles** desde el punto de montaje `/mnt/raid5` (`/mnt/raid10` para G2). Intenta leer su contenido.
    7. **Verifica el estado del arreglo RAID** y describe el estado.


### ✅ Evaluación del Laboratorio

**Prepara un informe individual que incluya:**

- Capturas de pantalla de los comandos exactos que utilizaste en cada paso.
- Capturas de pantalla que muestren la creación del RAID 5 (RAID 10 para G2), la simulación del fallo y el estado del arreglo después del fallo.
- Una descripción detallada de lo que observaste al intentar acceder a los archivos después de la falla simulada.
- Una explicación de cómo RAID 5 (RAID 10 para G2) permite la tolerancia a fallos en base al concepto de paridad distribuida.
- Breve resumen de las lecciones aprendidas sobre cómo el RAID 5 (RAID 10 para G2) proporciona redundancia y rendimiento, y una reflexión sobre la importancia de estas características en un entorno de producción.