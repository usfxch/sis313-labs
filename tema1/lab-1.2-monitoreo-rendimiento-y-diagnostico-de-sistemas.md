# Laboratorio 1.2: Monitoreo, Rendimiento y Diagnóstico de Sistemas

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## 🎯 Objetivos del Laboratorio

- **Monitorear y diagnosticar** el estado de salud del servidor (CPU, RAM, Disco y Red) en tiempo real utilizando herramientas avanzadas de terminal.

- **Gestionar el almacenamiento avanzado** mediante el particionamiento manual, formateo de diversos sistemas de archivos y la implementación de memoria SWAP.

- **Configurar conectividad de red profesional** estableciendo direccionamiento IP estático y gestionando la exposición de servicios mediante reenvío de puertos.

- **Evaluar la resiliencia del sistema** ejecutando pruebas de estrés y analizando registros (logs) de auditoría para la resolución de incidentes.

- **Automatizar el mantenimiento preventivo** y la administración de procesos para garantizar la continuidad operativa de la infraestructura.

## 🛠️ Sección 1: Preparación de la Infraestructura Virtual

En este laboratorio no empezaremos de cero. Utilizaremos el trabajo realizado en la sesión anterior para simular un escenario de **escalabilidad y mantenimiento**. Para ello, prepararemos el "hardware virtual" de nuestro servidor.

**1.0. Clonación de la Máquina Virtual**

Para preservar el estado de nuestro Laboratorio 1.1 y trabajar sobre una base segura, lo primero es crear una copia exacta.

1. Asegúrate de que la VM `SIS313-Lab1.1` esté **completamente apagada**.

2. Haz clic derecho sobre ella y selecciona **Clonar**.

3. **Nombre:** Cambia el nombre a `SIS313-Lab1.2`.

4. **Política de dirección MAC:** Selecciona **"Generar nuevas direcciones MAC para todas las tarjetas de red"**. (Esto evita conflictos de red si ambas máquinas llegaran a encenderse).

5. **Tipo de clonación:** Selecciona **Clonación completa**.

6. Haz clic en **Terminar/Finalizar**. A partir de ahora, trabajaremos únicamente en la nueva VM.

**1.1. Simulación de Almacenamiento Adicional**

En entornos de producción, los datos de los usuarios y el área de intercambio (SWAP) suelen residir en discos físicos distintos al del Sistema Operativo para mejorar el rendimiento y la seguridad. Vamos a añadir una nueva unidad:

1. Selecciona la VM `SIS313-Lab1.2` y haz clic en el botón naranja de **Configuración**.

2. Ve al apartado de **Almacenamiento** en el menú de la izquierda.

3. En el árbol de almacenamiento, busca el **Controlador: SATA**.

4. Haz clic en el icono de **Añadir Disco Duro** (el icono de un pequeño disco con un signo `+` verde).

5. Se abrirá un cuadro de diálogo: elige **Crear**.

6. Sigue el asistente manteniendo las opciones por defecto:

    - **Tipo:** VDI (VirtualBox Disk Image).

    - **Detalle:** Reservado dinámicamente.

    - **Tamaño:** Ajusta el nombre a Disco_Laboratorio_1.2 y el tamaño a **5.00 GB**.

7. Al finalizar, comprueba que bajo el Controlador SATA aparezcan tres elementos: la unidad óptica (ISO), el disco principal (20 GB) y el nuevo disco (5 GB).

**1.2. Configuración de Red Dual (Modo Puente)**

Para este laboratorio, nuestro servidor tendrá "dos tarjetas de red". La primera nos da salida a Internet (NAT) y la segunda nos permitirá integrarnos de forma transparente en la red local física.

1. Dentro de **Configuración**, ve al apartado de **Red**.

2. Haz clic en la pestaña **Adaptador 2**.

3. Marca la casilla **Habilitar adaptador de red**.

4. En "Conectado a:", selecciona **Adaptador Puente (Bridge)**.

5. **Importante:** En el campo "Nombre", despliega la lista y selecciona el adaptador que tu computadora real esté usando para internet (usualmente dirá algo como Wireless, Wi-Fi o Ethernet). Esto permitirá que tu VM obtenga una IP de tu router real.

6. Haz clic en **Aceptar**.

**1.3. Reenvío de Puertos (Servicio Web HTTP)**

Configuraremos un acceso directo para que, al escribir una dirección en el navegador de tu computadora física, la petición llegue directo al servidor Nginx de la VM.

1. En la misma ventana de **Red**, regresa a la pestaña **Adaptador 1**.

2. Asegúrate de que esté conectado a **NAT** y despliega la sección **Avanzadas**.

3. Haz clic en el botón **Reenvío de puertos**.

4. Añade una nueva regla con el botón `+` y completa los datos exactamente así:

    | Nombre | Protocolo | IP anfitrión | Puerto anfitrión | IP invitado | Puerto invitado |
    | - | - | - | - | - | - |
    | HTTP | TCP | | 8080 | 10.0.2.15 | 80 |

## 🖥️ Sección 2: Práctica Guiada (Administración y Diagnóstico)

En esta sección, pasaremos de la configuración física a la gestión lógica del sistema operativo. Cada comando es una herramienta de diagnóstico que un administrador de infraestructura utiliza para garantizar la disponibilidad.

**2.1. Diagnóstico de Hardware y Sistema Base**

Antes de realizar cambios, debemos conocer nuestra plataforma.

1. **Identificación del Sistema:**

    - `uname -a`: Muestra la versión del kernel y la arquitectura.

        ```bash
        uname -a
        ```

    - `hostnamectl`: Verifica el nombre del nodo. Cámbialo con `sudo`. 
        
        ```bash
        sudo hostnamectl set-hostname nodo-monitoreo-usfx
        ```

2. **Exploración de Hardware:**

    - `lscpu`: Visualiza los núcleos y arquitectura del procesador.

        ```bash
        lscpu
        ```

    - `lsblk`: Lista los bloques de almacenamiento. Identifica el nuevo disco de 5GB (probablemente aparece como `sdb` o `sdc`).

        ```bash
        lsblk
        ```

**2.2. Gestión de Almacenamiento: Particiones, SWAP y Persistencia**

Un servidor profesional separa sus datos para evitar que un llenado de disco bloquee el sistema operativo.

1. **Particionamiento Manual con `fdisk`:**

    - Entra al disco y asegúrate de que sea el disco de 5GB.

        ```bash
        sudo fdisk /dev/sdb
        ```

    - **Ejercicio:** Crear 3 particiones:

        - `n` -> `p` -> `1` -> Tamaño: `+1.5G` (Para Archivos A).

        - `n` -> `p` -> `2` -> Tamaño: `+1.5G` (Para Archivos B).

        - `n` -> `p` -> `3` -> Tamaño: `Enter` (El resto para SWAP).

    - **Cambiar tipo de SWAP:** Presiona `t`, selecciona la partición `3` y usa el código `82`.

    - Guarda cambios con `w`.

2. **Formateo y Activación:**

    ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```
    > Formatea la primera con sistema de archivos Linux estándar.

    ```bash
    sudo mkfs.xfs /dev/sdb2
    ```
    > Formatea la segunda con XFS (alto rendimiento).

    ```bash
    sudo mkswap /dev/sdb3
    ```
    > Prepara el área de intercambio.

    ```bash
    sudo swapon /dev/sdb3
    ```
    > Activa la SWAP inmediatamente.

3. **Montaje Persistente:**

    - Crea los puntos de montaje:
        ```bash
        sudo mkdir -p /mnt/datosA /mnt/datosB
        ```

    - Edita el archivo /etc/fstab:
        ```bash
        sudo nano /etc/fstab
        ```

    - Añade las líneas al final para que el montaje sobreviva a reinicios:

        ```
        /dev/sdb1  /mnt/datosA  ext4  defaults  0  2
        /dev/sdb2  /mnt/datosB  xfs   defaults  0  2
        /dev/sdb3  none         swap  sw        0  0
        ```

**2.3. Configuración de Red Avanzada (Netplan)**

Configuraremos la interfaz tipo puente con una IP fija para que el servidor sea siempre localizable.

1. Identifica el nombre de la nueva interfaz (ej. `enp0s8`) con `ip addr`.

    ```bash
    ip addr
    ```

2. Edita la configuración:

    ```bash
    sudo nano /etc/netplan/01-netcfg.yaml
    ```

3. Aplica una configuración similar a esta (ajustando a tu red local):

    ```yaml
    network:
    version: 2
    ethernets:
        enp0s8:
        dhcp4: no
        addresses: [192.168.1.100/24] # Usa una IP libre de tu red real
        gateway4: 192.168.1.1
        nameservers:
            addresses: [8.8.8.8, 1.1.1.1]
    ```

4. Aplica los cambios: 
    ```bash
    sudo netplan apply
    ```

**2.4. Instalación de Software y Repositorios de Terceros**

A veces el software oficial no es suficiente y necesitamos repositorios PPA (Personal Package Archives).

1. **Agregar Repositorio:** 

    ```bash
    sudo add-apt-repository ppa:pitti/proctools -y
    ```

2. **Actualizar e Instalar:** 

    ```bash
    sudo apt update && sudo apt install ncdu nload -y
    ```

    - `ncdu`: Herramienta visual para ver qué carpetas ocupan más espacio.

    - `nload`: Monitor de tráfico de red en tiempo real.

**2.5. Monitoreo de Rendimiento y Procesos**

¿Cómo saber si el servidor está sufriendo?

1. **Memoria y CPU:**

    - `free -m`: Verifica que la nueva SWAP aparezca sumada a la RAM.

        ```bash
        free -m
        ```

    - `htop`: Visualiza el uso de CPU. Identifica procesos con mucho consumo.

        ```bash
        htop
        ```

    - `vmstat 1 5`: Reporta estadísticas en tiempo real sobre los recursos del sistema. 

        ```bash
        vmstat 1 5
        ```
        > Observa la entrada/salida y el uso de memoria virtual cada segundo.

2. **Gestión de Procesos:**

    - `ps` (Process Status) muestra una instantánea de los procesos activos, permitiendo identificar su PID (ID de proceso).
        
        ```bash
        ps -aux | grep nginx
        ```
        > Localiza el ID de proceso (PID) de Nginx.
    
    - `kill` envía señales a dichos procesos (generalmente terminarlos) usando su PID.

        ```bash
        sudo kill -HUP [PID]
        ```
        > Reinicia un proceso sin apagarlo (recarga configuración).

        ```bash
        sudo kill -9 [PID]
        ```
        > Finaliza el proceso de manera forzada.

**2.6. Auditoría, Logs y Pruebas de Estrés**

Simularemos una falla crítica para aprender a leer los registros.

1. **Visualización de Logs:**

    ```bash
    journalctl -p err
    ```
    > Muestra solo los errores del sistema desde el último arranque.

    ```bash
    tail -n 20 /var/log/auth.log
    ```
    > Revisa los últimos intentos de acceso al servidor.

2. **Prueba de Estrés:**

    - Instala el paquete `stress`:
    
        ```bash
        sudo apt install stress
        ```

    - **Ejercicio de carga:** Ejecuta:

        ```bash
        stress --cpu 2 --io 1 --vm 1 --vm-bytes 128M --timeout 60s
        ```

    - En otra terminal abierta con `htop`, observa cómo cambian las barras de color y el "Load Average".

3. **Prueba de Escritura (Rendimiento de Disco):**

    ```bash
    dd if=/dev/zero of=/mnt/datosA/testfile bs=1G count=1 oflag=dsync
    ```
    > Esto creará un archivo de 1GB y te dirá la velocidad real (MB/s) de tu almacenamiento.

**2.7. Automatización con Cron**

Programaremos una tarea de limpieza automática.

1. Escribe:

    ```bash
    sudo crontab -e.
    ```

2. Añade una regla para que el sistema limpie los logs temporales todos los días a las 03:00 AM:

    ```
    0 3 * * * rm -rf /var/tmp/*
    ```

## 🏗️ Sección 3: Desafío de Diagnóstico y Resolución de Problemas

**Contexto:** Usted ha sido contratado como administrador de servidores en el Centro de Datos de la Universidad. Se le ha reportado que uno de los nodos de servicios (su VM) está experimentando lentitud intermitente y errores de "Disco Lleno" en aplicaciones críticas. Su misión es estabilizar el sistema, ampliar la memoria virtual y asegurar que los servicios web sean accesibles desde la red externa.

**El Escenario Técnico**

Deberá realizar las siguientes tareas de forma autónoma. No se proporcionarán los comandos exactos, ya que debe deducirlos de la `Sección 2`.

**Ejercicio 1: Gestión de Almacenamiento y Cuotas**

El sistema actual se está quedando sin espacio para los registros de auditoría.

1. **Particionamiento:** Utilice el disco de 5GB añadido en la Sección 1. Cree una partición de **2GB** con sistema de archivos **ext4**.

2. **Montaje Crítico:** Monte esta partición en la ruta `/var/log/servicios`.

3. **Persistencia:** Asegúrese de que, si el servidor se reinicia, esta partición se monte automáticamente (edite `/etc/fstab`).

4. **Verificación de Inodos:** Ejecute un comando que muestre el uso de inodos de esta nueva partición y explique en su informe por qué es importante monitorear este valor además del espacio en GB.

**Ejercicio 2: Mitigación de Crisis de Memoria**

Se anticipa una carga masiva de usuarios y la RAM física es insuficiente.

1. **Ampliación de SWAP:** Utilice el espacio restante del disco de 5GB (aprox. 3GB) para crear y activar una nueva área de intercambio (SWAP).

2. **Validación:** Utilice una herramienta de monitoreo para demostrar que la SWAP total del sistema ha incrementado su capacidad.

**Ejercicio 3: Diagnóstico de Red y Exposición de Servicios**

El departamento de desarrollo necesita acceder al servidor desde fuera de la red NAT.

1. **IP Estática:** Configure la interfaz en **Modo Puente (Bridge)** con una dirección IP fija que pertenezca al segmento de su red local (ej. si su PC tiene la `192.168.1.15`, asigne al servidor la `192.168.1.200`).

2. **Prueba de Fuego:** Desde el navegador de su computadora física (Windows/Mac), acceda a la dirección `http://127.0.0.1:8080`. Debe visualizar la página de bienvenida de Nginx.

**Ejercicio 4: Simulación de Carga y Auditoría**

Debe identificar qué sucede cuando el servidor es atacado o saturado.

1. **Prueba de Estrés:** Utilice la herramienta `stress` para saturar la CPU al 100% durante **2 minutos**.

2. **Identificación de "Culpables":** Mientras el sistema está bajo estrés, identifique el proceso de `stress` con el comando `ps` y determine cuál es su PID y qué porcentaje de CPU está consumiendo.

3. **Filtro de Logs:** Busque en el historial del sistema (`journalctl`) todos los eventos de prioridad "Advertencia" (Warning) o superior que hayan ocurrido en los últimos 30 minutos y expórtelos a un archivo llamado `incidencias.txt`.

### 📄 Instrucciones para los Entregables

Para que el laboratorio sea válido, el estudiante debe presentar un documento PDF con los resultados del desafío. Cada tarea debe incluir:

1. **Captura de pantalla clara:** Donde se vea el comando ejecutado y el resultado exitoso.

2. **Descripción técnica:** Un párrafo breve (2-3 líneas) explicando qué hizo el comando y cómo soluciona el problema planteado en el escenario.

3. **Cuadro de Métricas:**

    - Muestre el resultado de `free -m` antes y después de activar la SWAP.

    - Muestre el "Load Average" del sistema durante la prueba de estrés (obtenido de `top` o `uptime`).