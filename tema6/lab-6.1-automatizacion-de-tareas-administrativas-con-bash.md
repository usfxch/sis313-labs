# Laboratorio 6.1: Automatización de Tareas Administrativas con Bash

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Dominar los fundamentos de Bash Scripting** (variables, argumentos, condicionales, bucles, pipes).

- **Automatizar tareas repetitivas** de administración como gestión de usuarios, monitoreo de servicios y análisis de logs.

- **Desarrollar scripts reutilizables** para el despliegue de software y la verificación de estado del sistema.

- **Construir interfaces CLI** mediante menús interactivos que faciliten la ejecución controlada de tareas.

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando **2 Máquinas Virtuales (VMs)** con **Ubuntu Server 24.04 LTS**.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.40.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab6.1-Admin` | `admin` | **SERVIDOR DE ADMINISTRACIÓN** (Scripts, Cron, Menú) | NAT (Internet) + Red Interna | `192.168.40.2` |
    | `Lab6.1-Target` | `target` | **SERVIDOR OBJETIVO** (Pruebas de usuarios, servicios) | Red Interna | `192.168.40.3` |

    - **Gateway (GW):** La interfaz interna de la VM `Lab6.1-Admin` actuará como puerta de enlace para la VM `Target`.

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab6_1`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab6.1-Admin (Administración):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab6_1`).

        - **VM Lab6.1-Target (Objetivo):**

            - Adaptador 1: **Red Interna** (`Red_Lab6_1`).

    3. **Reenvío de Puertos (Port Forwarding) en VM Lab6.1-Admin (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab6.1-Admin` para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP | 127.0.0.1 | **2222** | 10.0.2.15 | 22 | Acceso Remoto |

3. **Preparar directorios de trabajo en la VM Admin**

    ```bash
    sudo mkdir -p /opt/admin_scripts
    sudo mkdir -p /var/backups/data_center
    sudo chmod 755 /opt/admin_scripts
    ```

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Configuración de Red Estática

1. **Configurar IP estática en la VM Admin:**

    ```bash
    sudo nano /etc/netplan/50-cloud-init.yaml
    ```

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s3:
          dhcp4: true
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.40.2/29
          nameservers:
            addresses:
              - 8.8.8.8
    ```

    ```bash
    sudo netplan apply
    ```

2. **Configurar IP estática en la VM Target:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.40.3/29
          nameservers:
            addresses:
              - 192.168.40.2
          routes:
            - to: default
              via: 192.168.40.2
    ```

3. **Habilitar reenvío de paquetes en la VM Admin:**

    ```bash
    sudo nano /etc/sysctl.conf
    # Asegurar: net.ipv4.ip_forward=1
    sudo sysctl -p
    sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

4. **Configurar acceso SSH por clave desde Admin a Target:**

    En la VM Admin, generar un par de claves:

    ```bash
    ssh-keygen -t ed25519 -C "admin@lab61" -f ~/.ssh/id_lab61
    ```

    Copiar la clave pública a la VM Target:

    ```bash
    ssh-copy-id -i ~/.ssh/id_lab61.pub usuario@192.168.40.3
    ```

    Probar conexión sin contraseña:

    ```bash
    ssh -i ~/.ssh/id_lab61 usuario@192.168.40.3
    ```

### Ejercicio 2: Script de Bienvenida y Log (Variables y Redireccionamiento)

1. **Crear el script `01_intro.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/01_intro.sh
    ```

    ```bash
    #!/bin/bash
    # Script de bienvenida y registro de acceso

    LOG_FILE="/tmp/admin_access.log"
    NOMBRE=$1
    ROL=$2

    echo "========================================="
    echo "¡Bienvenido, $NOMBRE! Su rol es $ROL."
    echo "========================================="

    # Redireccionamiento: añadir al log con fecha y usuario actual
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Usuario del sistema: $USER. Nombre: $NOMBRE, Rol: $ROL." >> $LOG_FILE

    echo "Último registro añadido a $LOG_FILE:"
    tail -n 1 $LOG_FILE
    ```

    > **Explicación:**
    > - `$1` y `$2` son argumentos posicionales pasados al script.
    > - `$USER` es una variable de entorno que contiene el usuario que ejecuta el script.
    > - `>>` redirige la salida añadiendo (append) al archivo sin sobrescribir.

2. **Otorgar permisos de ejecución y probar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/01_intro.sh
    /opt/admin_scripts/01_intro.sh "Juan Perez" "Administrador"
    /opt/admin_scripts/01_intro.sh "Ana Lopez" "Soporte"
    ```

3. **Verificar el contenido del log:**

    ```bash
    cat /tmp/admin_access.log
    ```

### Ejercicio 3: Script de Verificación de Archivos y Directorios (Condicionales)

1. **Crear el script `02_check.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/02_check.sh
    ```

    ```bash
    #!/bin/bash
    # Verificación rápida de archivos y directorios críticos

    LOG_FILE="/tmp/admin_access.log"
    DIR_WEB="/var/www/html"

    # Verificar si el archivo de log existe
    if [ -f "$LOG_FILE" ]; then
        echo "[OK] El archivo de log $LOG_FILE existe."
    else
        echo "[ALERTA] El archivo de log NO fue encontrado."
    fi

    # Verificar si el directorio web existe
    if [ -d "$DIR_WEB" ]; then
        echo "[OK] El directorio web $DIR_WEB existe."
    else
        echo "[ERROR] El directorio web $DIR_WEB no existe. Creándolo..."
        sudo mkdir -p "$DIR_WEB"
        echo "[OK] Directorio creado."
    fi

    # Verificar uso de disco (alerta si supera 85%)
    USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//g')
    if [ "$USAGE" -gt 85 ]; then
        echo "[CRITICO] Uso de disco: $USAGE%. Limpieza requerida."
    else
        echo "[OK] Uso de disco: $USAGE%."
    fi
    ```

    > **Explicación:**
    > - `[ -f ]` verifica si es un archivo regular.
    > - `[ -d ]` verifica si es un directorio.
    > - `awk 'NR==2 {print $5}'` extrae la quinta columna de la segunda línea (uso de disco).

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/02_check.sh
    /opt/admin_scripts/02_check.sh
    ```

### Ejercicio 4: Procesamiento de Puertos con Pipes y Filtros

1. **Crear el script `03_pipes.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/03_pipes.sh
    ```

    ```bash
    #!/bin/bash
    # Top 5 puertos TCP en escucha

    echo "Top 5 puertos TCP más utilizados:"
    sudo ss -tuln | grep 'tcp ' | awk '{print $5}' | cut -d':' -f2 | sort | uniq -c | sort -nr | head -n 5
    ```

    > **Explicación del pipe:**
    > - `ss -tuln`: muestra sockets TCP/UDP en escucha.
    > - `grep 'tcp '`: filtra solo líneas TCP.
    > - `awk '{print $5}'`: extrae la columna 5 (dirección:puerto).
    > - `cut -d':' -f2`: separa por `:` y toma el puerto.
    > - `sort | uniq -c | sort -nr`: cuenta ocurrencias y ordena descendente.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/03_pipes.sh
    /opt/admin_scripts/03_pipes.sh
    ```

### Ejercicio 5: Iteración y Análisis de Logs (Bucles `for` y `while`)

1. **Crear el script `04_summarize_logs.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/04_summarize_logs.sh
    ```

    ```bash
    #!/bin/bash
    # Análisis de logs de Nginx con bucles

    LOG_DIR="/var/log/nginx/"
    ACCESS_LOG="/var/log/nginx/access.log"
    STATUS_COUNT=0

    echo "--- 1. Conteo de archivos log (bucle FOR) ---"

    for LOG_FILE in $LOG_DIR*.log; do
        if [ -f "$LOG_FILE" ]; then
            LINE_COUNT=$(wc -l < "$LOG_FILE")
            echo "  [FOR] $(basename $LOG_FILE): $LINE_COUNT líneas"
        fi
    done

    echo -e "\n--- 2. Análisis de peticiones 200 (bucle WHILE) ---"

    if [ -f "$ACCESS_LOG" ]; then
        while read LINE; do
            if echo "$LINE" | grep -q " 200 "; then
                STATUS_COUNT=$((STATUS_COUNT + 1))
            fi
        done < "$ACCESS_LOG"
        echo "  [WHILE] Total peticiones HTTP 200: $STATUS_COUNT"
    else
        echo "  [ERROR] $ACCESS_LOG no encontrado"
    fi
    ```

    > **Explicación:**
    > - `for`: itera sobre archivos que coinciden con el patrón.
    > - `while read LINE`: procesa el archivo línea por línea sin cargarlo completamente en memoria.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/04_summarize_logs.sh
    sudo /opt/admin_scripts/04_summarize_logs.sh
    ```

### Ejercicio 6: Gestión Masiva de Usuarios desde CSV

1. **En la VM Admin, crear el archivo CSV:**

    ```bash
    sudo bash -c 'echo "ana_sistemas,sistemas" > /opt/admin_scripts/usuarios.csv'
    sudo bash -c 'echo "luis_soporte,soporte" >> /opt/admin_scripts/usuarios.csv'
    sudo bash -c 'echo "eva_sistemas,sistemas" >> /opt/admin_scripts/usuarios.csv'
    sudo bash -c 'echo "carlos_redes,redes" >> /opt/admin_scripts/usuarios.csv'
    ```

2. **Crear el script `05_user_manager.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/05_user_manager.sh
    ```

    ```bash
    #!/bin/bash
    # Gestión masiva de usuarios y grupos desde CSV

    CSV_FILE="/opt/admin_scripts/usuarios.csv"

    if [ ! -f "$CSV_FILE" ]; then
        echo "[ERROR] Archivo $CSV_FILE no encontrado."
        exit 1
    fi

    cat "$CSV_FILE" | while IFS=',' read -r USERNAME GROUPNAME; do
        # Limpiar espacios en blanco
        GROUPNAME=$(echo "$GROUPNAME" | tr -d '[:space:]')
        USERNAME=$(echo "$USERNAME" | tr -d '[:space:]')

        # Crear grupo si no existe
        if ! grep -q "^$GROUPNAME:" /etc/group; then
            sudo groupadd "$GROUPNAME"
            echo "[OK] Grupo '$GROUPNAME' creado."
        fi

        # Crear usuario si no existe
        if ! id "$USERNAME" &>/dev/null; then
            sudo useradd -m -g "$GROUPNAME" -s /bin/bash "$USERNAME"
            echo "[OK] Usuario '$USERNAME' creado (grupo: $GROUPNAME)."
        else
            echo "[INFO] Usuario '$USERNAME' ya existe."
        fi
    done
    ```

    > **Explicación:**
    > - `IFS=','`: define la coma como separador de campos.
    > - `read -r USERNAME GROUPNAME`: lee dos columnas por línea.
    > - `id "$USERNAME" &>/dev/null`: verifica silenciosamente si el usuario existe.

3. **Ejecutar en la VM Admin:**

    ```bash
    sudo chmod +x /opt/admin_scripts/05_user_manager.sh
    sudo /opt/admin_scripts/05_user_manager.sh
    ```

4. **Verificar usuarios creados:**

    ```bash
    grep -E "sistemas|soporte|redes" /etc/group
    id ana_sistemas
    id luis_soporte
    ```

5. **Replicar en la VM Target vía SSH:**

    Copiar el CSV y el script a la VM Target:

    ```bash
    scp -i ~/.ssh/id_lab61 /opt/admin_scripts/usuarios.csv /opt/admin_scripts/05_user_manager.sh usuario@192.168.40.3:/tmp/
    ```

    Ejecutar remotamente:

    ```bash
    ssh -i ~/.ssh/id_lab61 usuario@192.168.40.3 "sudo bash /tmp/05_user_manager.sh"
    ```

    > **Explicación:** Esto demuestra cómo automatizar tareas en múltiples servidores usando SSH sin contraseña.

### Ejercicio 7: Despliegue Desatendido de Servicios

1. **Crear el script `deploy_nginx.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/deploy_nginx.sh
    ```

    ```bash
    #!/bin/bash
    # Despliegue desatendido de Nginx y configuración base

    echo "[INFO] Iniciando despliegue de Nginx..."

    # Actualizar repositorios e instalar
    sudo apt update
    sudo apt install nginx -y

    # Habilitar e iniciar el servicio
    sudo systemctl enable --now nginx

    # Verificar estado
    if systemctl is-active --quiet nginx; then
        echo "[OK] Nginx instalado y activo."
    else
        echo "[ERROR] Nginx no pudo iniciarse."
        exit 1
    fi

    # Crear página de prueba
    sudo bash -c 'echo "<h1>Servidor desplegado automaticamente</h1>" > /var/www/html/index.html'
    sudo bash -c 'echo "<p>Fecha de despliegue: '"$(date)'"</p>" >> /var/www/html/index.html'

    echo "[OK] Despliegue completado."
    ```

    > **Explicación:**
    > - `apt update && apt install -y`: actualiza e instala sin interacción del usuario.
    > - `systemctl enable --now`: habilita el servicio para iniciar automáticamente en el arranque y lo inicia de inmediato.
    > - Este patrón puede extenderse para desplegar múltiples servicios en servidores nuevos.

2. **Ejecutar en la VM Target vía SSH:**

    ```bash
    sudo chmod +x /opt/admin_scripts/deploy_nginx.sh
    scp -i ~/.ssh/id_lab61 /opt/admin_scripts/deploy_nginx.sh usuario@192.168.40.3:/tmp/
    ssh -i ~/.ssh/id_lab61 usuario@192.168.40.3 "sudo bash /tmp/deploy_nginx.sh"
    ```

3. **Verificar desde la VM Admin que el servicio responde:**

    ```bash
    curl http://192.168.40.3
    ```

### Ejercicio 8: Limpieza Automatizada de Logs del Sistema

1. **Crear el script `log_cleanup.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/log_cleanup.sh
    ```

    ```bash
    #!/bin/bash
    # Limpieza automatizada de logs antiguos del sistema

    DIAS=30
    LOG_DIRS="/var/log /var/log/nginx /var/log/apache2"
    REPORTE="/tmp/cleanup_report.log"

    echo "[INFO] Iniciando limpieza de logs mayores a $DIAS dias..." | sudo tee "$REPORTE"

    for DIR in $LOG_DIRS; do
        if [ -d "$DIR" ]; then
            COUNT=$(find "$DIR" -type f -name "*.log*" -mtime +$DIAS | wc -l)
            if [ "$COUNT" -gt 0 ]; then
                find "$DIR" -type f -name "*.log*" -mtime +$DIAS -delete
                echo "[OK] $DIR: $COUNT logs eliminados." | sudo tee -a "$REPORTE"
            else
                echo "[INFO] $DIR: No hay logs antiguos para eliminar." | sudo tee -a "$REPORTE"
            fi
        fi
    done

    echo "[INFO] Limpieza finalizada. Reporte: $REPORTE" | sudo tee -a "$REPORTE"
    ```

    > **Explicación:**
    > - `find -type f -name "*.log*" -mtime +30`: encuentra archivos de log con más de 30 días de antigüedad.
    > - El bucle `for` permite aplicar la limpieza a múltiples directorios de logs.
    > - Se genera un reporte para auditoría.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/log_cleanup.sh
    sudo /opt/admin_scripts/log_cleanup.sh
    ```

3. **Verificar el reporte:**

    ```bash
    cat /tmp/cleanup_report.log
    ```

### Ejercicio 9: Rollback de Usuarios Creados

1. **Crear el script `user_cleanup.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/user_cleanup.sh
    ```

    ```bash
    #!/bin/bash
    # Rollback: eliminar usuarios y grupos creados desde el CSV

    CSV_FILE="/opt/admin_scripts/usuarios.csv"

    if [ ! -f "$CSV_FILE" ]; then
        echo "[ERROR] Archivo $CSV_FILE no encontrado."
        exit 1
    fi

    echo "⚠️  Este script eliminará los usuarios y grupos definidos en $CSV_FILE"
    read -p "¿Continuar? (si/no): " CONFIRM

    if [ "$CONFIRM" != "si" ]; then
        echo "Cancelado."
        exit 0
    fi

    cat "$CSV_FILE" | while IFS=',' read -r USERNAME GROUPNAME; do
        USERNAME=$(echo "$USERNAME" | tr -d '[:space:]')
        GROUPNAME=$(echo "$GROUPNAME" | tr -d '[:space:]')

        if id "$USERNAME" &>/dev/null; then
            sudo userdel -r "$USERNAME"
            echo "[OK] Usuario '$USERNAME' eliminado."
        else
            echo "[INFO] Usuario '$USERNAME' no existe."
        fi

        if grep -q "^$GROUPNAME:" /etc/group; then
            # Verificar que no queden usuarios en el grupo
            MEMBERS=$(grep "^$GROUPNAME:" /etc/group | cut -d':' -f4)
            if [ -z "$MEMBERS" ]; then
                sudo groupdel "$GROUPNAME"
                echo "[OK] Grupo '$GROUPNAME' eliminado."
            else
                echo "[INFO] Grupo '$GROUPNAME' tiene miembros, no se elimina."
            fi
        fi
    done
    ```

    > **Explicación:**
    > - `userdel -r`: elimina el usuario y su directorio home.
    > - `groupdel`: elimina el grupo solo si está vacío.
    > - El script incluye una confirmación interactiva para evitar eliminaciones accidentales.

2. **Ejecutar (opcional, para revertir el Ejercicio 6):**

    ```bash
    sudo chmod +x /opt/admin_scripts/user_cleanup.sh
    sudo /opt/admin_scripts/user_cleanup.sh
    ```

3. **Verificar que los usuarios fueron eliminados:**

    ```bash
    id ana_sistemas  # Debe indicar que no existe
    grep "sistemas" /etc/group
    ```

### Ejercicio 10: Health Check Automático de Servicios y Disco

1. **Crear el script `06_check_system.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/06_check_system.sh
    ```

    ```bash
    #!/bin/bash
    # Health check de servicios críticos y recursos

    LOGFILE="/var/log/system_check.log"
    sudo touch $LOGFILE

    # 1. Verificar estado de Nginx
    if ! systemctl is-active --quiet nginx; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - ALERTA: Nginx inactivo. Reiniciando..." | sudo tee -a $LOGFILE > /dev/null
        sudo systemctl restart nginx
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - OK: Nginx activo." | sudo tee -a $LOGFILE > /dev/null
    fi

    # 2. Verificar uso de disco
    USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//g')
    THRESHOLD=85

    if [ "$USAGE" -gt "$THRESHOLD" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - CRITICO: Disco al $USAGE%." | sudo tee -a $LOGFILE > /dev/null
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - OK: Disco al $USAGE%." | sudo tee -a $LOGFILE > /dev/null
    fi

    # 3. Verificar memoria disponible
    MEM_AVAILABLE=$(free | grep Mem | awk '{print $7}')
    echo "$(date '+%Y-%m-%d %H:%M:%S') - INFO: Memoria disponible: ${MEM_AVAILABLE}KB." | sudo tee -a $LOGFILE > /dev/null

    echo "----------------------------------------"
    echo "Últimas entradas del log:"
    sudo tail -n 5 $LOGFILE
    ```

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/admin_scripts/06_check_system.sh
    sudo /opt/admin_scripts/06_check_system.sh
    ```

### Ejercicio 11: Menú Interactivo de Administración

1. **Crear el usuario `menu` en la VM Admin:**

    ```bash
    sudo useradd -m -s /bin/bash menu
    sudo passwd menu
    ```

2. **Crear el script `07_admin_menu.sh`:**

    ```bash
    sudo nano /opt/admin_scripts/07_admin_menu.sh
    ```

    ```bash
    #!/bin/bash
    # Menú interactivo de administración

    CHECK_SCRIPT="/opt/admin_scripts/06_check_system.sh"
    USER_SCRIPT="/opt/admin_scripts/05_user_manager.sh"

    while true; do
        clear
        echo "=========================================="
        echo "  M E N Ú   D E   A D M I N I S T R A C I Ó N"
        echo "=========================================="
        echo "1) Health Check (Servicios y Disco)"
        echo "2) Gestión Masiva de Usuarios (CSV)"
        echo "3) Ver Logs de Acceso"
        echo "4) Salir"
        echo "=========================================="
        read -p "Ingrese una opción [1-4]: " opcion

        case $opcion in
            1)
                echo "Ejecutando health check..."
                sudo bash "$CHECK_SCRIPT"
                read -p "Presione Enter para continuar..."
                ;;
            2)
                echo "Ejecutando gestión de usuarios..."
                sudo bash "$USER_SCRIPT"
                read -p "Presione Enter para continuar..."
                ;;
            3)
                echo "Mostrando últimos registros..."
                if [ -f /tmp/admin_access.log ]; then
                    tail -n 10 /tmp/admin_access.log
                else
                    echo "No hay registros aún."
                fi
                read -p "Presione Enter para continuar..."
                ;;
            4)
                echo "Saliendo del menú. ¡Hasta pronto!"
                break
                ;;
            *)
                echo "Opción inválida. Intente de nuevo."
                read -p "Presione Enter para continuar..."
                ;;
        esac
    done
    ```

    > **Explicación:**
    > - `while true`: bucle infinito hasta que el usuario elija salir.
    > - `case`: ejecuta diferentes bloques según la opción seleccionada.
    > - `clear`: limpia la pantalla para una mejor experiencia de usuario.

3. **Configurar ejecución automática al iniciar sesión con el usuario `menu`:**

    ```bash
    sudo nano /home/menu/.bashrc
    ```

    Agregar al final:

    ```bash
    # Menú de administración automático
    if [ -f "/opt/admin_scripts/07_admin_menu.sh" ]; then
        bash /opt/admin_scripts/07_admin_menu.sh
    fi
    ```

4. **Probar:**

    Cambiar al usuario `menu`:

    ```bash
    su - menu
    ```

    > **Resultado esperado:** Al iniciar sesión, debe aparecer automáticamente el menú interactivo.

## ⚙️ Sección 3: Práctica en Grupo (Red entre Pares)

El objetivo es que cada grupo de **2 estudiantes** intercambie scripts y valide la automatización en las VMs del compañero.

### 1. Roles y Asignación por Grupo

| Integrante | Rol | VM a Configurar | Requisitos de Red |
| - | - | - | - |
| **Integrante 1** | **Administrador Principal** | VM Admin (scripts, menú, health checks) | Adaptador en modo **Puente (Bridge)** |
| **Integrante 2** | **Administrador Secundario** | VM Target (pruebas de usuarios, servicios) | Adaptador en modo **Puente (Bridge)** |

### 2. Tareas del Integrante 1

1. **Configurar la VM en modo Puente** con IP estática.

2. **Crear un script personalizado** (`grupoX_deploy.sh`) que:
    - Acepte como argumento el nombre del grupo.
    - Cree un directorio `/var/www/grupoX`.
    - Genere un archivo `index.html` con los nombres de los integrantes.
    - Registre la acción en `/tmp/deploy.log`.

    Ejemplo:

    ```bash
    #!/bin/bash
    GRUPO=$1
    INTEGRANTE1=$2
    INTEGRANTE2=$3
    DIR="/var/www/$GRUPO"

    if [ -z "$GRUPO" ]; then
        echo "Uso: $0 <nombre_grupo> <integrante1> <integrante2>"
        exit 1
    fi

    sudo mkdir -p "$DIR"
    sudo bash -c "echo '<h1>Grupo $GRUPO</h1><p>$INTEGRANTE1 y $INTEGRANTE2</p>' > $DIR/index.html"
    echo "$(date): Deploy de $GRUPO completado." | sudo tee -a /tmp/deploy.log
    ```

3. **Compartir el script** con el Integrante 2 (mediante `scp` o archivo compartido).

### 3. Tareas del Integrante 2

1. **Configurar la VM en modo Puente** con IP estática.

2. **Ejecutar el script recibido** del Integrante 1 en su VM.

3. **Validar que el directorio y archivo se crearon correctamente.**

4. **Crear un script de health check cruzado** que verifique desde su VM que el servidor web del compañero responde:

    ```bash
    #!/bin/bash
    IP_COMPAÑERO="192.168.x.x"
    if curl -s -o /dev/null -w "%{http_code}" http://$IP_COMPAÑERO | grep -q "200"; then
        echo "[OK] Servidor web del compañero responde correctamente."
    else
        echo "[ALERTA] Servidor web del compañero NO responde."
    fi
    ```

### 4. Reto Grupal: Script Combinado

Juntos, crear un script `inventory.sh` que:

1. Recorra una lista de IPs (las VMs de ambos integrantes) guardadas en un archivo `servers.txt`.
2. Para cada servidor, verifique:
    - Si responde al ping.
    - Si el puerto 22 está abierto.
    - Si Nginx está activo (vía SSH remoto).
3. Genere un reporte en `/tmp/inventory_report.txt`.

Ejemplo de `servers.txt`:

```
192.168.40.2
192.168.40.3
192.168.x.x
```

Ejemplo de `inventory.sh`:

```bash
#!/bin/bash
while read IP; do
    echo "Servidor: $IP"
    ping -c 1 -W 2 "$IP" > /dev/null && echo "  [OK] Ping" || echo "  [FAIL] Ping"
    nc -z -w 2 "$IP" 22 && echo "  [OK] SSH(22)" || echo "  [FAIL] SSH(22)"
done < servers.txt
```

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual (scripts Bash, menú, gestión de usuarios): 35 pts
- Desarrollo de la práctica grupal (deploy cruzado, health check remoto, inventory): 35 pts
- Informe detallado con capturas de pantalla, código de scripts y análisis: 30 pts
