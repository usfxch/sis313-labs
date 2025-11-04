# Laboratorio 6: Automatización de Administración y Backups Seguros

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

- **Automatizar Administración**: Crear scripts Bash para la **gestión masiva** de usuarios/grupos y el **monitoreo** de servicios/disco.

- **Backup Remoto Seguro:** Desarrollar un script para el backup (`mysqldump`) de la DB (VM 3) usando **SSH/clave privada** y asegurar la **retención de datos**.

- **Planificación con Cron:** Programar la ejecución automática y periódica de los scripts de backup y monitoreo usando `cron`.

- **Menú de Control Interactivo:** Implementar un menú Bash (`case`/`while`) para la **ejecución manual** y controlada de las tareas de administración.


## 🛠️ Preparación del Entorno Virtual

Se utilizará la infraestructura de 3 capas ya configurada en el Laboratorio 5.1 (Hardening), centrándose en el **Proxy (VM 1)** y el **Servidor de Base de Datos (VM 3)**.

| VM | Rol | Tarea Principal en Lab 6 | Herramientas |
| - | - | - | - |
| **VM 1 (Proxy/LB)** | Servidor de Administración | Scripts de gestión y monitoreo. | Bash, Cron |
| **VM 3 (DB)** | Servidor de Base de Datos | **Target** del backup: Contiene la Base de Datos a respaldar. | Bash, `mysqldump`, Cron |

### Requerimientos Iniciales

1. **Conexión SSH Asegurada:** Asegurarse de poder acceder a ambas VMs usando la **clave privada** (sin contraseña) y a través del puerto modificado (ej., 2222), según el Laboratorio 5.1.

2. **Usuario de Backup:** En la **VM 3 (DB)**, verificar que exista un archivo de credenciales seguro (`~/.my.cnf`) o un usuario de MariaDB con los permisos de `SELECT` y `LOCK TABLES` para realizar el `mysqldump`.

3. **Directorio de Destino:** Crear un directorio para los scripts y los backups en la **VM 1** (Proxy):

    ```bash
    # En VM 1
    sudo mkdir -p /opt/admin_scripts
    sudo mkdir -p /var/backups/data_center
    ```

4. **Creación del Usuario menu (VM 1):** Este usuario se utilizará exclusivamente para el Ejercicio 6.

    ```bash
    sudo useradd -m -s /bin/bash menu
    sudo passwd menu
    # (Asignar una contraseña segura)
    ```

## 💻 Práctica Guiada

### Ejercicio 1: Fundamentos de Bash para la Administración

El objetivo es practicar los elementos esenciales de Bash Scripting que se usarán en los ejercicios posteriores.

1. **Script de Bienvenida y Log (Variables y Redireccionamiento):**

    - **Función:** Crea un script que acepte **dos argumentos** (`$1` como nombre, `$2` como rol) y registre la información en un archivo de log, incluyendo la fecha y el usuario (`$USER`).

    - **Comandos a utilizar:** Uso de `echo`, `$1`, `$2`, `date`, `$USER`, y `>>`.

    - **Script:**

        ```bash
        # En VM 1: Crear /opt/admin_scripts/01_intro.sh
        #!/bin/bash

        # 1. Variables de entorno y argumentos
        LOG_FILE="/tmp/admin_access.log"
        NOMBRE=$1
        ROL=$2

        echo "========================================="
        echo "¡Bienvenido, $NOMBRE! Su rol es $ROL."
        echo "========================================="

        # 2. Redireccionamiento (Añadir al log)
        echo "$(date +%Y-%m-%d\ %H:%M:%S) - Usuario: $USER. Nombre: $NOMBRE, Rol: $ROL. Script ejecutado." >> $LOG_FILE

        # 3. Mostrar el log
        echo "Último registro añadido a $LOG_FILE:"
        tail -n 1 $LOG_FILE
        ```

    - **Ejecución del script:**

        ```bash
        sudo chmod +x /opt/admin_scripts/01_intro.sh
        ```

        ```bash
        /opt/admin_scripts/01_intro.sh Juan Administrador
        ```

2. **Script de Verificación Rápida (Condicionales y Códigos de Salida):**

    - **Función:** Verifica si el archivo del ejercicio anterior existe y si un directorio específico (`/var/www/html`) existe. Reporta el estado.

    - **Comando a utilizar:** Uso de `if`, `[ -f ]` (archivo), `[ -d ]` (directorio), y la sentencia `else`.

    - **Script:**

        ```bash
        # En VM 1: Crear /opt/admin_scripts/02_check.sh
        #!/bin/bash

        LOG_FILE="/tmp/admin_access.log"
        DIR_WEB="/var/www/html"

        # 1. Condicional para chequear archivo
        if [ -f "$LOG_FILE" ]; then
            echo "INFO: El archivo de log existe."
        else
            echo "ALERTA: El archivo de log NO fue encontrado."
        fi

        # 2. Condicional para chequear directorio
        if [ -d "$DIR_WEB" ]; then
            echo "INFO: El directorio web $DIR_WEB existe."
        else
            echo "ERROR: El directorio web $DIR_WEB no existe. Crear con 'mkdir -p $DIR_WEB'."
            exit 1
        fi
        ```
    
    - **Ejecución del script:**

        ```bash
        sudo chmod +x /opt/admin_scripts/02_check.sh
        ```

        ```bash
        /opt/admin_scripts/02_check.sh
        ```
    
3. **Script de Procesamiento de Puertos (Pipes y Tuberías):**

    - **Función:** Muestra los 5 puertos TCP que están siendo utilizados activamente en el sistema (abiertos o escuchando).

    - **Comandos a utilizar:** Uso de netstat (o ss), Pipe (|), grep (para filtrar TCP), awk (para cortar campos) y sort o uniq para contar.

    - **Script:**

        ```bash
        # En VM 1: Crear /opt/admin_scripts/03_pipes.sh
        #!/bin/bash

        echo "Top 5 puertos TCP más usados/escuchados:"

        # 1. netstat -tuln muestra puertos TCP/UDP (t=TCP, u=UDP, l=listening, n=numeric)
        # 2. grep 'tcp ' filtra solo las líneas TCP
        # 3. awk '{print $4}' obtiene la cuarta columna (dirección:puerto)
        # 4. cut -d':' -f2 extrae solo el número de puerto
        # 5. sort | uniq -c cuenta las ocurrencias
        # 6. sort -nr ordena numéricamente y de forma reversa (mayor primero)
        # 7. head -n 5 muestra solo los 5 primeros

        sudo netstat -tuln | grep 'tcp ' | awk '{print $4}' | cut -d':' -f2 | sort | uniq -c | sort -nr | head -n 5
        ```

    - **Ejecución del script:**

        ```bash
        sudo chmod +x /opt/admin_scripts/03_pipes.sh
        ```

        ```bash
        /opt/admin_scripts/03_pipes.sh
        ```

4. **Iteración y Procesamiento de Logs (Bucles `for` y `while`):**

    - **Función:** Simula la búsqueda de archivos de log en un directorio (`/var/log/nginx/`), cuenta las líneas de cada uno (usando `for`), y luego procesa un archivo específico línea por línea (usando `while`) para contar las peticiones con código 200.

    - **Estructura:** Bucle `for` para archivos; `while read` para contenido.

    - **Script:**

        ```bash
        # En VM 1: Crear /opt/admin_scripts/04_summarize_logs.sh
        #!/bin/bash

        LOG_DIR="/var/log/nginx/"
        ACCESS_LOG="/var/log/nginx/access.log"
        STATUS_COUNT=0

        echo "--- 1. Conteo de Archivos Log (Bucle FOR) ---"

        # 1. Bucle FOR: Itera sobre todos los archivos .log en el directorio
        for LOG_FILE in $LOG_DIR*.log; do
            if [ -f "$LOG_FILE" ]; then
                LINE_COUNT=$(wc -l < "$LOG_FILE")
                echo " [FOR] Archivo: $(basename $LOG_FILE) tiene $LINE_COUNT líneas."
            fi
        done

        echo -e "\n--- 2. Análisis de Peticiones Exitosas (Bucle WHILE) ---"

        # 2. Bucle WHILE: Procesa el archivo access.log línea por línea
        # Esto es eficiente para archivos grandes.
        if [ -f "$ACCESS_LOG" ]; then
            # Redirecciona el archivo al bucle while
            while read LINE; do
                # 3. Condicional: Busca el código de estado HTTP 200
                if echo "$LINE" | grep -q " 200 "; then
                    STATUS_COUNT=$((STATUS_COUNT + 1))
                fi
            done < $ACCESS_LOG
            # Se imprime el resultado fuera del bucle while (después de la subshell)
            echo " [WHILE] Total de peticiones 200 (éxito) encontradas: $STATUS_COUNT"
        else
            echo " [ERROR] Archivo $ACCESS_LOG no encontrado para análisis."
        fi
        ```

    - **Comandos a utilizar:** Uso de estructuras iterativas `for` y `while`.

    - **Ejecución del script:**

        ```bash
        sudo chmod +x /opt/admin_scripts/04_summarize_logs.sh
        ```

        ```bash
        /opt/admin_scripts/04_summarize_logs.sh
        ```

### Ejercicio 2: Gestión de Grupos y Usuarios Compleja

El objetivo es crear un script que no solo añada usuarios, sino que también asegure que los grupos a los que pertenecen existan previamente.

**Script de Gestión de Grupos y Usuarios (`group_user_manager.sh`):**

- **Función:** Lee un archivo CSV (`usuarios.csv`) que define el usuario y el grupo principal. El script debe chequear la existencia del grupo antes de crear al usuario.

- **Estructura:** Uso de `if`, `groupadd`, `useradd` y separación de campos (`IFS`).

    ```bash
    # En VM 1: Crear usuarios.csv (Formato: USUARIO,GRUPO)
    sudo bash -c 'echo "ana_sistemas,sistemas" > /opt/admin_scripts/usuarios.csv'
    sudo bash -c 'echo "luis_soporte,soporte" >> /opt/admin_scripts/usuarios.csv'
    sudo bash -c 'echo "eva_sistemas,sistemas" >> /opt/admin_scripts/usuarios.csv'
    ```
    > Guarda el archivo en el directorio `/opt/admin_scripts` con el nombre `usuarios.csv`.

    ```bash
    # En VM 1: Crear el script group_user_manager.sh
    #!/bin/bash

    # 1. Bucle para procesar el CSV. IFS=',' usa la coma como separador.
    cat usuarios.csv | while IFS=',' read -r USERNAME GROUPNAME; do
        GROUPNAME=$(echo $GROUPNAME | tr -d '[:space:]') # Limpiar espacios en el grupo

        # 2. Chequeo de existencia del GRUPO
        if ! grep -q "^$GROUPNAME:" /etc/group; then
            echo "El grupo $GROUPNAME no existe. Creándolo..."
            sudo groupadd "$GROUPNAME"
        fi

        # 3. Creación del USUARIO en el grupo
        if ! id "$USERNAME" &>/dev/null; then
            sudo useradd -m -g "$GROUPNAME" -s /bin/bash "$USERNAME"
            echo "Usuario $USERNAME creado y asignado al grupo $GROUPNAME."
        else
            echo "Usuario $USERNAME ya existe."
        fi
    done
    ```

    > Guarda el archivo en el directorio `/opt/admin_scripts` con el nombre `user_manager.sh`.

- **Ejecución del script:**

    ```bash
    sudo chmod +x /opt/admin_scripts/user_manager.sh
    ```

    ```bash
    sudo /opt/admin_scripts/user_manager.sh
        ```

### Ejercicio 3: Automatización de Health Check y Alerta

El objetivo es crear un script de monitoreo para el chequeo de espacio en disco utilizando Pipes (`|`).

**Script de Health Check (check_system.sh):**

- **Función:** Monitorea el servicio Nginx y el uso del sistema de archivos raíz (`/`). Si el disco supera el 85%, se registra una alerta de saturación.

- **Estructura:** Uso de Pipes, `awk` o `sed` para extraer datos numéricos y comparaciones (`-gt`).

- **Script:**

    ```bash
    # En VM 1: Crear el script check_system.sh
    #!/bin/bash
    LOGFILE="/var/log/system_check.log"

    # 1. Chequeo de Servicio Nginx (Repetición del Lab 5.1)
    systemctl is-active --quiet nginx
    if [ $? -ne 0 ]; then
        echo "$(date): ERROR: Nginx inactivo. Reiniciando..." >> $LOGFILE
        sudo systemctl restart nginx
    fi

    # 2. Chequeo de Uso de Disco (Uso de pipe y awk)
    # df -h / | awk 'NR==2 {print $5}' extrae el porcentaje de la segunda línea, quinta columna.
    USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//g')
    THRESHOLD=85

    if [ "$USAGE" -gt "$THRESHOLD" ]; then
        echo "$(date): ALERTA CRÍTICA: Uso de disco al $USAGE%. Limpieza requerida." >> $LOGFILE
    else
        echo "$(date): INFO: Disco OK ($USAGE%)." >> $LOGFILE
    fi
    ```

- **Ejecución del script:**

    ```bash
    sudo chmod +x /opt/admin_scripts/system_check.sh
    ```

    ```bash
    sudo /opt/admin_scripts/system_check.sh
    ```

### Ejercicio 4: Script de Backup Automático y Seguro de Base de Datos

El objetivo es automatizar la copia de seguridad de la Base de Datos (VM 3) al Proxy (VM 1) de forma segura.

**Script de Backup Remoto (`db_backup.sh`):**

- **Ubicación:** Crear y ejecutar en la **VM 1 (Proxy/LB)**.

- **Función:** Utilizar `mysqldump` de forma remota y `ssh` para ejecutar el dump localmente en la VM 3 y comprimir el resultado.

    - Modificar el archivo `/etc/.my.cnf` en el VM 3.

        ```ini
        [mysqldump]
        # O [client] si la cuenta se usa para otras herramientas MySQL
        user = app_user             # El usuario que solo tiene permisos de SELECT y LOCK TABLES
        password = TuContraseñaFuerteAquí 
        host = 127.0.0.1            # Se conecta a sí mismo (localhost)
        # port = 3306               # (Opcional) Si se usa el puerto estándar

        # Opción recomendada para copias de seguridad consistentes con InnoDB
        single-transaction
        ```

- **Script:**

    ```bash
    # En VM 1: Crear el script db_backup.sh
    #!/bin/bash

    # Variables de Configuración
    DB_HOST="192.168.200.4" # IP de la VM 3 (Servidor DB)
    DB_NAME="db_movies"
    SSH_USER="usr_movies"
    BACKUP_DIR="/var/backups/data_center"
    FILE_DATE=$(date +%Y%m%d_%H%M)
    PORT_SSH="2222" # Puerto SSH modificado (Lab 5.1)

    # 1. Comandos de mysqldump a ser ejecutados en la VM 3 via SSH
    # Nota: El usuario de DB debe tener credenciales seguras configuradas en .my.cnf en la VM 3
    echo "Iniciando backup remoto de $DB_NAME en $DB_HOST..."

    ssh -p $PORT_SSH $SSH_USER@$DB_HOST "mysqldump --defaults-file=/etc/.my.cnf $DB_NAME" | gzip > $BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz

    if [ $? -eq 0 ]; then
        echo "Backup completado con éxito: $BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz"

        # 2. Retención: Eliminar backups más antiguos de 7 días (Seguridad y limpieza)
        find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete
        echo "Limpieza de archivos antiguos (más de 7 días) completada."
    else
        echo "ERROR: Falló el backup remoto de la base de datos."
    fi
    ```

- **Ejecución del script:**

    ```bash
    sudo chmod +x /opt/admin_scripts/db_backup.sh
    ```

    ```bash
    sudo /opt/admin_scripts/db_backup.sh

### Ejercicio 5: Rotación de Logs Simplificada

El objetivo es usar la función de **manejo de archivos** para simular una rotación de logs manual, asegurando el conocimiento de `tar`.

**Script de Rotación y Compresión (`log_rotate_simple.sh`):**

- **Función:** Comprime los logs antiguos de Nginx, los archiva y luego vacía el archivo de log original.

- **Estructura:** Uso de `mv`, `tar`, `echo >` (vaciar archivo).

- **Script:**

    ```bash
    # En VM 1: Crear el script log_rotate_simple.sh
    #!/bin/bash
    LOG_NGINX="/var/log/nginx/access.log"
    LOG_DIR_ARCHIVE="/var/log/nginx/archive"

    # Asegurar que el directorio de archivo exista
    sudo mkdir -p $LOG_DIR_ARCHIVE

    if [ -f "$LOG_NGINX" ]; then
        # 1. Renombrar el archivo log actual
        sudo mv $LOG_NGINX $LOG_NGINX.$(date +%Y%m%d)

        # 2. Comprimir y archivar el log renombrado
        sudo tar -czf $LOG_DIR_ARCHIVE/access-$(date +%Y%m%d).tar.gz $LOG_NGINX.$(date +%Y%m%d)

        # 3. Vaciar y recrear el archivo de log original (o notificar a Nginx)
        sudo touch $LOG_NGINX
        sudo echo "$(date): [LOG ROTATION] Log rotado con éxito." > $LOG_NGINX

        # 4. Eliminar el archivo temporal sin comprimir
        sudo rm $LOG_NGINX.$(date +%Y%m%d)

        echo "Log de Nginx rotado y archivado en $LOG_DIR_ARCHIVE."
    fi
    ```

- **Ejecución del script:**

    ```bash
    sudo chmod +x /opt/admin_scripts/log_rotate_simple.sh
    ```

    ```bash
    sudo /opt/admin_scripts/log_rotate_simple.sh
    ```

### Ejercicio 6: Menú de Administración Interactivo

El objetivo es utilizar estructuras `while` y `case` para crear una interfaz de línea de comandos (CLI) simple. Se ejecutará automáticamente al iniciar sesión con el usuario `menu`.

1. **Script del Menú (`admin_menu.sh`):**

    - **Ubicación:** `/opt/admin_scripts/admin_menu.sh` en VM 1.

    - **Función:** Muestra opciones para ejecutar los scripts creados manualmente.

    - **Estructura:** Bucle `while`, sentencia `case`, `select`.

    - **Script:**

        ```bash
        # En VM 1: Crear el script admin_menu.sh
        #!/bin/bash

        # Rutas a los scripts (deben tener permisos de ejecución para el usuario 'menu')
        CHECK_SCRIPT="/opt/admin_scripts/check_system.sh"
        BACKUP_SCRIPT="/opt/admin_scripts/db_backup.sh"

        # 1. Función para mostrar el menú
        mostrar_menu() {
            clear
            echo "=========================================="
            echo "  M E N Ú  D E  A D M I N I S T R A C I Ó N"
            echo "=========================================="
            echo "1) Checkup de Servicios y Disco (Ejecuta Health Check)"
            echo "2) Ejecutar Backup Manual de DB (Ejecuta Backup Script)"
            echo "3) Salir"
            echo "=========================================="
        }

        # 2. Bucle principal
        while true; do
            mostrar_menu
            read -p "Ingrese una opción [1-3]: " opcion

            case $opcion in
                1)
                    echo "Ejecutando chequeo del sistema..."
                    bash $CHECK_SCRIPT
                    read -p "Presione Enter para continuar..."
                    ;;
                2)
                    echo "Ejecutando backup de la base de datos..."
                    bash $BACKUP_SCRIPT
                    read -p "Presione Enter para continuar..."
                    ;;
                3)
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

    - **Ejecución del script:**

        ```bash
        sudo chmod +x /opt/admin_scripts/log_rotate_simple.sh
        ```

        ```bash
        sudo /opt/admin_scripts/log_rotate_simple.sh
        ```

2. **Configuración de Login Automático del Menú:**

    - El usuario `menu` debe ejecutar el menú automáticamente al hacer `login`.

        ```bash
        # En VM 1: Editar el archivo de perfil del usuario 'menu'
        sudo nano /home/menu/.bashrc
        ```

        ```bash
        # Añadir estas líneas al FINAL del archivo
        if [ -f "/opt/admin_scripts/admin_menu.sh" ]; then
            /opt/admin_scripts/admin_menu.sh
            # Opcional: si el menú se cierra, que cierre la sesión
            # exit
        fi
        ```

### Ejercicio 7: Planificación de la Ejecución con Cron

El objetivo es asegurar que el *script* de backup se ejecute de forma diaria y automática.

1. **Configurar Cron:**

    Ejecutar el comando para editar el crontab de un usuario *sudoer* (ya que el script usa rutas de sistema y potencialmente credenciales elevadas):

    ```bash
    # En VM 1
    sudo crontab -e
    ```

2. **Añadir la Tarea Programada:**

    Agregar la siguiente línea al final del archivo `crontab`.

    ```bash
    # Backup diario de la base de datos (cada medianoche a las 00:00)
    0 0 * * * /opt/scripts/db_backup.sh > /dev/null 2>&1
    ```

    **Explicación:**

    - `0 0 * * *`: Se ejecuta en el minuto 0 de la hora 0, de cada día del mes, de cada mes, de cada día de la semana (diariamente a medianoche).

    - `> /dev/null 2>&1`: Redirecciona la salida estándar (`> /dev/null`) y los errores (`2>&1`) a un 'agujero negro', asegurando que `cron` no envíe correos electrónicos por cada ejecución exitosa o con errores menores.

3. **Verificación:**

    - Verificar que la tarea ha sido cargada:

        ```bash
        sudo crontab -l
        ```
    - **Opcional:** Para una verificación inmediata, cambiar la hora a una que ocurra en los siguientes 5 minutos (ej., `*/5 * * * *`).

### Ejercicio 8 (Opcional): Script de Rollback y Limpieza Total 🧹

**Objetivo:** Desarrollar un script (`cleanup_lab6.sh`) que sea capaz de deshacer todos los cambios realizados durante los Ejercicios 1, 2, 4, 5 y 6 de este laboratorio (sin afectar los cambios de hardening del Lab 5.1).

**Requisitos Específicos:**

1. El script debe **eliminar todos los usuarios** creados (`juan_perez`, `ana_sistemas`, etc.) y **borrar los grupos** asociados (ej., `sistemas`, `soporte`).

2. Debe **eliminar todos los archivos de log y backup** generados en `/var/backups/data_center`, `/var/log/nginx/archive` y otros.

3. Debe **eliminar la configuración de** `cron` del usuario `sudoer` (la tarea de backup diario) y **deshabilitar el menú interactivo** (revirtiendo el cambio en el archivo `.bashrc` del usuario `menu`).

4. Debe **eliminar** los archivos de los scripts creados en `/opt/admin_scripts/`.

5. El script debe incluir una **pausa de confirmación** antes de iniciar la limpieza total, avisando al usuario de las consecuencias.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica: 70 pts 

- Informe detallado con capturas de pantalla que demuestren los ejercicios realizados: 30 pts