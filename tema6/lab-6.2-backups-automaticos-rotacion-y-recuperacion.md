# Laboratorio 6.2: Backups Automáticos, Rotación y Recuperación

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Desarrollar scripts de backup** para archivos (`tar`) y bases de datos (`mysqldump`).

- **Implementar backups remotos seguros** mediante SSH con compresión (`gzip`).

- **Aplicar políticas de retención** de datos y rotación de logs automatizadas.

- **Planificar tareas periódicas** con `cron` para la ejecución desatendida de backups.

- **Verificar la recuperación** de datos y servicios, documentando el RTO (Recovery Time Objective).

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando **2 Máquinas Virtuales (VMs)** con **Ubuntu Server 24.04 LTS**.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.50.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab6.2-Backup` | `backup` | **SERVIDOR DE BACKUPS** (Scripts, Cron, Almacenamiento) | NAT (Internet) + Red Interna | `192.168.50.2` |
    | `Lab6.2-DB` | `db` | **SERVIDOR DE BASE DE DATOS** (MariaDB, Archivos Web) | Red Interna | `192.168.50.3` |

    - **Gateway (GW):** La interfaz interna de la VM `Lab6.2-Backup` actuará como puerta de enlace para la VM `DB`.

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab6_2`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab6.2-Backup (Servidor de Backups):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab6_2`).

        - **VM Lab6.2-DB (Base de Datos):**

            - Adaptador 1: **Red Interna** (`Red_Lab6_2`).

    3. **Reenvío de Puertos (Port Forwarding) en VM Lab6.2-Backup (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab6.2-Backup` para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP | 127.0.0.1 | **2222** | 10.0.2.15 | 22 | Acceso Remoto |

3. **Preparar directorios de trabajo**

    En la VM Backup:

    ```bash
    sudo mkdir -p /opt/backup_scripts
    sudo mkdir -p /var/backups/data_center
    sudo mkdir -p /var/backups/files
    sudo chmod 755 /opt/backup_scripts
    ```

    En la VM DB:

    ```bash
    sudo mkdir -p /var/www/html
    sudo mkdir -p /var/log/nginx/archive
    ```

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Configuración de Red y Acceso SSH por Clave

1. **Configurar IP estática en la VM Backup:**

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
            - 192.168.50.2/29
          nameservers:
            addresses:
              - 8.8.8.8
    ```

    ```bash
    sudo netplan apply
    ```

2. **Configurar IP estática en la VM DB:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.50.3/29
          nameservers:
            addresses:
              - 192.168.50.2
          routes:
            - to: default
              via: 192.168.50.2
    ```

3. **Habilitar reenvío de paquetes en la VM Backup:**

    ```bash
    sudo nano /etc/sysctl.conf
    # Asegurar: net.ipv4.ip_forward=1
    sudo sysctl -p
    sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

4. **Configurar acceso SSH sin contraseña desde Backup a DB:**

    En la VM Backup:

    ```bash
    ssh-keygen -t ed25519 -C "backup@lab62" -f ~/.ssh/id_backup
    ssh-copy-id -i ~/.ssh/id_backup.pub usuario@192.168.50.3
    ```

    Probar:

    ```bash
    ssh -i ~/.ssh/id_backup usuario@192.168.50.3 "hostname"
    ```

    > **Explicación:** El backup remoto requiere una relación de confianza por clave SSH para ser completamente automatizado.

### Ejercicio 2: Preparar el Servidor de Base de Datos

1. **En la VM DB, instalar MariaDB y Nginx:**

    ```bash
    sudo apt update
    sudo apt install mariadb-server nginx -y
    sudo systemctl enable --now mariadb
    sudo systemctl enable --now nginx
    ```

2. **Crear una base de datos y tabla de prueba:**

    ```bash
    sudo mysql -e "CREATE DATABASE IF NOT EXISTS lab62_db;"
    sudo mysql -e "USE lab62_db; CREATE TABLE IF NOT EXISTS productos (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(100), precio DECIMAL(10,2));"
    sudo mysql -e "USE lab62_db; INSERT INTO productos (nombre, precio) VALUES ('Laptop', 1200.00), ('Mouse', 25.50), ('Teclado', 45.00);"
    ```

3. **Verificar los datos insertados:**

    ```bash
    sudo mysql -e "SELECT * FROM lab62_db.productos;"
    ```

4. **Crear archivo de credenciales seguro `.my.cnf`:**

    ```bash
    sudo nano /etc/.my.cnf
    ```

    ```ini
    [mysqldump]
    user = root
    password = tu_password_root
    host = 127.0.0.1
    single-transaction
    ```

    > **Explicación:** `single-transaction` garantiza un backup consistente en tablas InnoDB sin bloquear escrituras. Guardar credenciales en un archivo evita exponer contraseñas en la línea de comandos o en scripts.

5. **Proteger el archivo de credenciales:**

    ```bash
    sudo chmod 600 /etc/.my.cnf
    sudo chown root:root /etc/.my.cnf
    ```

6. **Crear contenido web de prueba:**

    ```bash
    sudo bash -c 'echo "<h1>Sitio de Prueba - Lab 6.2</h1>" > /var/www/html/index.html'
    sudo bash -c 'echo "<p>Datos importantes del laboratorio</p>" >> /var/www/html/index.html'
    sudo mkdir -p /var/www/html/docs
    sudo bash -c 'echo "Documento confidencial" > /var/www/html/docs/secreto.txt'
    ```

### Ejercicio 3: Backup Local de Archivos con `tar`

1. **En la VM Backup, crear el script `file_backup.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/file_backup.sh
    ```

    ```bash
    #!/bin/bash
    # Backup local de archivos web con tar y gzip

    DESTINO="/var/backups/files"
    FECHA=$(date +%Y%m%d_%H%M)
    DIR_FUENTE="/var/www/html"

    sudo mkdir -p "$DESTINO"

    # Verificar que el directorio fuente existe
    if [ ! -d "$DIR_FUENTE" ]; then
        echo "[ERROR] Directorio fuente $DIR_FUENTE no existe."
        exit 1
    fi

    # Crear backup comprimido
    sudo tar -czf "$DESTINO/web-$FECHA.tar.gz" -C "$(dirname $DIR_FUENTE)" "$(basename $DIR_FUENTE)"

    if [ $? -eq 0 ]; then
        echo "[OK] Backup creado: $DESTINO/web-$FECHA.tar.gz"
        ls -lh "$DESTINO/web-$FECHA.tar.gz"
    else
        echo "[ERROR] Falló la creación del backup."
        exit 1
    fi
    ```

    > **Explicación:**
    > - `tar -czf`: `-c` crear, `-z` comprimir con gzip, `-f` especificar archivo.
    > - `-C $(dirname)`: cambia al directorio padre antes de archivar, evitando rutas absolutas.

2. **Ejecutar en la VM Backup (archivos locales):**

    ```bash
    sudo chmod +x /opt/backup_scripts/file_backup.sh
    sudo /opt/backup_scripts/file_backup.sh
    ```

    > **Nota:** Como el directorio `/var/www/html` está vacío en la VM Backup, el resultado será un backup pequeño. En la práctica grupal se hará backup remoto de la VM DB.

### Ejercicio 4: Backup Remoto de Base de Datos mediante SSH

1. **En la VM Backup, crear el script `db_backup.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/db_backup.sh
    ```

    ```bash
    #!/bin/bash
    # Backup remoto de base de datos via SSH + mysqldump + gzip

    DB_HOST="192.168.50.3"
    DB_NAME="lab62_db"
    SSH_USER="usuario"
    BACKUP_DIR="/var/backups/data_center"
    FILE_DATE=$(date +%Y%m%d_%H%M)
    SSH_KEY="/home/usuario/.ssh/id_backup"

    sudo mkdir -p "$BACKUP_DIR"

    echo "[INFO] Iniciando backup remoto de $DB_NAME desde $DB_HOST..."

    # Ejecutar mysqldump en la VM DB via SSH y comprimir localmente
    ssh -i "$SSH_KEY" "$SSH_USER@$DB_HOST" \
        "sudo mysqldump --defaults-file=/etc/.my.cnf $DB_NAME" \
        | gzip | sudo tee "$BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz" > /dev/null

    if [ $? -eq 0 ]; then
        echo "[OK] Backup completado: $BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz"
        ls -lh "$BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz"
    else
        echo "[ERROR] Falló el backup remoto."
        exit 1
    fi
    ```

    > **Explicación:**
    > - El comando `mysqldump` se ejecuta **remotamente** en la VM DB.
    > - La salida se transmite por SSH de forma cifrada.
    > - `gzip` comprime el flujo de datos en tiempo real.
    > - `sudo tee` permite escribir en un directorio protegido desde un pipe.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/backup_scripts/db_backup.sh
    sudo /opt/backup_scripts/db_backup.sh
    ```

3. **Verificar el contenido del backup sin descomprimir completamente:**

    ```bash
    zcat /var/backups/data_center/lab62_db-*.sql.gz | head -20
    ```

### Ejercicio 5: Verificación de Integridad de Backups

Antes de confiar en un backup como única copia de seguridad, es fundamental verificar que no esté corrupto y que contenga los datos esperados.

1. **Crear el script `verify_backup.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/verify_backup.sh
    ```

    ```bash
    #!/bin/bash
    # Verificación de integridad de backups de base de datos

    BACKUP_DIR="/var/backups/data_center"
    LATEST=$(ls -t $BACKUP_DIR/lab62_db-*.sql.gz 2>/dev/null | head -n 1)

    if [ -z "$LATEST" ]; then
        echo "[ERROR] No se encontró ningún backup de lab62_db."
        exit 1
    fi

    echo "[INFO] Verificando backup: $(basename $LATEST)"

    # 1. Verificar que el archivo no esté vacío
    SIZE=$(stat -c%s "$LATEST")
    if [ "$SIZE" -eq 0 ]; then
        echo "[FALLO] El backup tiene tamaño 0 bytes."
        exit 1
    fi
    echo "[OK] Tamaño del backup: $SIZE bytes."

    # 2. Verificar que sea un gzip válido
    if gzip -t "$LATEST" 2>/dev/null; then
        echo "[OK] Archivo gzip válido."
    else
        echo "[FALLO] Archivo gzip corrupto."
        exit 1
    fi

    # 3. Verificar que contenga la estructura esperada
    if zcat "$LATEST" | grep -q "CREATE TABLE"; then
        echo "[OK] Contiene estructura de tablas (CREATE TABLE)."
    else
        echo "[ADVERTENCIA] No se encontró CREATE TABLE en el backup."
    fi

    # 4. Verificar que contenga datos insertados
    COUNT=$(zcat "$LATEST" | grep -c "INSERT INTO")
    if [ "$COUNT" -gt 0 ]; then
        echo "[OK] Contiene $COUNT sentencias INSERT INTO."
    else
        echo "[ADVERTENCIA] No se encontraron sentencias INSERT."
    fi

    echo "[INFO] Verificación completada exitosamente."
    ```

    > **Explicación:**
    > - `gzip -t`: testea la integridad del archivo comprimido sin extraerlo.
    > - `zcat | grep`: permite inspeccionar el contenido del backup sin generar archivos temporales.
    > - Verificar estructura (`CREATE TABLE`) y datos (`INSERT INTO`) garantiza que el backup es funcional.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/backup_scripts/verify_backup.sh
    sudo /opt/backup_scripts/verify_backup.sh
    ```

### Ejercicio 6: Reporte de Estado de Backups

1. **Crear el script `backup_report.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/backup_report.sh
    ```

    ```bash
    #!/bin/bash
    # Reporte de estado de backups con tamaños y fechas

    BACKUP_DIR="/var/backups/data_center"
    FILES_DIR="/var/backups/files"
    REPORTE="/tmp/backup_report.txt"

    echo "========================================" > "$REPORTE"
    echo "  REPORTE DE BACKUPS - $(date)" >> "$REPORTE"
    echo "========================================" >> "$REPORTE"
    echo "" >> "$REPORTE"

    echo "--- Backups de Base de Datos ---" >> "$REPORTE"
    if [ -d "$BACKUP_DIR" ] && [ "$(ls -A $BACKUP_DIR)" ]; then
        ls -lh "$BACKUP_DIR"/*.sql.gz 2>/dev/null | awk '{print $9, " | Tamaño:", $5, " | Fecha:", $6, $7, $8}' >> "$REPORTE"
        TOTAL_BD=$(ls -1 "$BACKUP_DIR"/*.sql.gz 2>/dev/null | wc -l)
        echo "Total: $TOTAL_BD backups" >> "$REPORTE"
    else
        echo "No hay backups de BD." >> "$REPORTE"
    fi

    echo "" >> "$REPORTE"
    echo "--- Backups de Archivos ---" >> "$REPORTE"
    if [ -d "$FILES_DIR" ] && [ "$(ls -A $FILES_DIR)" ]; then
        ls -lh "$FILES_DIR"/*.tar.gz 2>/dev/null | awk '{print $9, " | Tamaño:", $5, " | Fecha:", $6, $7, $8}' >> "$REPORTE"
        TOTAL_FILES=$(ls -1 "$FILES_DIR"/*.tar.gz 2>/dev/null | wc -l)
        echo "Total: $TOTAL_FILES backups" >> "$REPORTE"
    else
        echo "No hay backups de archivos." >> "$REPORTE"
    fi

    echo "" >> "$REPORTE"
    echo "--- Uso de Disco en /var/backups ---" >> "$REPORTE"
    df -h /var/backups >> "$REPORTE"

    cat "$REPORTE"
    ```

    > **Explicación:**
    > - `ls -t`: ordena por fecha de modificación (más reciente primero).
    > - `awk`: extrae y formatea columnas específicas (nombre, tamaño, fecha).
    > - El reporte unifica backups de BD, archivos y estado del disco en un solo documento.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/backup_scripts/backup_report.sh
    sudo /opt/backup_scripts/backup_report.sh
    ```

### Ejercicio 7: Rotación de Logs de Nginx

1. **En la VM DB, generar tráfico para crear logs:**

    ```bash
    for i in {1..20}; do curl -s http://localhost/ > /dev/null; done
    ```

    Verificar que exista el log:

    ```bash
    ls -lh /var/log/nginx/access.log
    wc -l /var/log/nginx/access.log
    ```

2. **En la VM DB, crear el script `log_rotate.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/log_rotate.sh
    ```

    ```bash
    #!/bin/bash
    # Rotación manual de logs de Nginx

    LOG_NGINX="/var/log/nginx/access.log"
    ARCHIVE="/var/log/nginx/archive"
    FECHA=$(date +%Y%m%d)

    sudo mkdir -p "$ARCHIVE"

    if [ -f "$LOG_NGINX" ]; then
        # 1. Renombrar log actual
        sudo mv "$LOG_NGINX" "$LOG_NGINX.$FECHA"

        # 2. Comprimir y archivar
        sudo tar -czf "$ARCHIVE/access-$FECHA.tar.gz" -C /var/log/nginx "access.log.$FECHA"

        # 3. Eliminar archivo temporal
        sudo rm "$LOG_NGINX.$FECHA"

        # 4. Recrear log vacío y notificar a Nginx
        sudo touch "$LOG_NGINX"
        sudo chown www-data:adm "$LOG_NGINX"
        sudo chmod 640 "$LOG_NGINX"

        echo "[OK] Log rotado y archivado: $ARCHIVE/access-$FECHA.tar.gz"
    else
        echo "[ERROR] Archivo $LOG_NGINX no encontrado."
        exit 1
    fi
    ```

    > **Explicación:**
    > - `mv`: renombra el log activo (Nginx puede seguir escribiendo en el descriptor abierto).
    > - `tar -czf`: comprime el log renombrado.
    > - `touch + chown + chmod`: recrea el archivo con permisos correctos para Nginx.

3. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/backup_scripts/log_rotate.sh
    sudo /opt/backup_scripts/log_rotate.sh
    ```

4. **Verificar resultado:**

    ```bash
    ls -lh /var/log/nginx/archive/
    ls -lh /var/log/nginx/access.log
    ```

### Ejercicio 8: Retención de Datos y Limpieza Automática

1. **Crear el script `cleanup_old_backups.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/cleanup_old_backups.sh
    ```

    ```bash
    #!/bin/bash
    # Política de retención: eliminar backups y archivos de log mayores a 7 días

    BACKUP_DIR="/var/backups/data_center"
    FILES_DIR="/var/backups/files"
    LOG_ARCHIVE="/var/log/nginx/archive"

    echo "[INFO] Aplicando política de retención (7 días)..."

    # Eliminar backups de BD antiguos
    if [ -d "$BACKUP_DIR" ]; then
        COUNT=$(find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 | wc -l)
        find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
        echo "[OK] $COUNT backups de BD eliminados."
    fi

    # Eliminar backups de archivos antiguos
    if [ -d "$FILES_DIR" ]; then
        COUNT=$(find "$FILES_DIR" -name "*.tar.gz" -mtime +7 | wc -l)
        find "$FILES_DIR" -name "*.tar.gz" -mtime +7 -delete
        echo "[OK] $COUNT backups de archivos eliminados."
    fi

    # Eliminar logs archivados antiguos
    if [ -d "$LOG_ARCHIVE" ]; then
        COUNT=$(find "$LOG_ARCHIVE" -name "*.tar.gz" -mtime +7 | wc -l)
        find "$LOG_ARCHIVE" -name "*.tar.gz" -mtime +7 -delete
        echo "[OK] $COUNT archivos de log eliminados."
    fi

    echo "[INFO] Limpieza completada."
    ```

    > **Explicación:**
    > - `find ... -mtime +7`: encuentra archivos modificados hace más de 7 días.
    > - `-delete`: elimina los archivos encontrados.
    > - La política 3-2-1 recomienda mantener copias en múltiples ubicaciones; aquí simulamos la retención local.

2. **Ejecutar:**

    ```bash
    sudo chmod +x /opt/backup_scripts/cleanup_old_backups.sh
    sudo /opt/backup_scripts/cleanup_old_backups.sh
    ```

### Ejercicio 9: Planificación con Cron

1. **En la VM Backup, configurar tareas programadas:**

    ```bash
    sudo crontab -e
    ```

    Agregar las siguientes líneas:

    ```bash
    # Backup de base de datos todos los días a las 00:00
    0 0 * * * /opt/backup_scripts/db_backup.sh > /dev/null 2>&1

    # Backup de archivos todos los domingos a las 02:00
    0 2 * * 0 /opt/backup_scripts/file_backup.sh > /dev/null 2>&1

    # Limpieza de backups antiguos todos los lunes a las 03:00
    0 3 * * 1 /opt/backup_scripts/cleanup_old_backups.sh > /dev/null 2>&1
    ```

    > **Explicación:**
    > - `0 0 * * *`: minuto 0, hora 0, todos los días, todos los meses, todos los días de la semana.
    > - `0 2 * * 0`: minuto 0, hora 2, día 0 (domingo).
    > - `> /dev/null 2>&1`: suprime salida estándar y errores para evitar spam de correo de cron.

2. **Verificar tareas cargadas:**

    ```bash
    sudo crontab -l
    ```

3. **(Opcional) Probar ejecución inmediata:**

    Cambiar temporalmente la línea del backup a cada 5 minutos:

    ```bash
    */5 * * * * /opt/backup_scripts/db_backup.sh > /dev/null 2>&1
    ```

    Esperar 5 minutos y verificar que se creó un nuevo backup:

    ```bash
    ls -lt /var/backups/data_center/
    ```

    > **Recuerda** volver a la configuración original después de la prueba.

### Ejercicio 10: Restauración de Archivos desde `tar`

1. **Verificar el contenido del backup sin extraer:**

    ```bash
    tar -tzf /var/backups/files/web-*.tar.gz | head -10
    ```

2. **Simular pérdida de datos y restaurar:**

    ```bash
    # Simular eliminación accidental
    sudo rm -rf /var/www/html/*
    echo "[SIMULACIÓN] Datos eliminados."
    ls -la /var/www/html/
    ```

3. **Restaurar desde el backup:**

    ```bash
    sudo tar -xzf /var/backups/files/web-*.tar.gz -C /var/www/
    ```

    > **Explicación:**
    > - `tar -xzf`: `-x` extraer, `-z` descomprimir gzip, `-f` archivo.
    > - `-C /var/www/`: extrae en el directorio destino.

4. **Verificar restauración:**

    ```bash
    ls -la /var/www/html/
    cat /var/www/html/index.html
    ```

### Ejercicio 11: Restauración de Base de Datos desde SQL

1. **En la VM DB, simular corrupción/pérdida de datos:**

    ```bash
    sudo mysql -e "DROP DATABASE lab62_db;"
    sudo mysql -e "SHOW DATABASES;"
    ```

2. **Restaurar la base de datos desde el backup:**

    ```bash
    # Descomprimir y restaurar
    zcat /var/backups/data_center/lab62_db-*.sql.gz | sudo mysql
    ```

    O en dos pasos:

    ```bash
    zcat /var/backups/data_center/lab62_db-*.sql.gz > /tmp/restore.sql
    sudo mysql < /tmp/restore.sql
    ```

3. **Verificar integridad de los datos restaurados:**

    ```bash
    sudo mysql -e "SHOW DATABASES;"
    sudo mysql -e "SELECT * FROM lab62_db.productos;"
    ```

    > **Resultado esperado:** La base de datos `lab62_db` y sus 3 registros deben haberse recuperado completamente.

4. **Calcular el RTO aproximado:**

    ```bash
    time zcat /var/backups/data_center/lab62_db-*.sql.gz | sudo mysql
    ```

    Documentar el tiempo transcurrido como el RTO del servicio de base de datos.

### Ejercicio 12: Script de Rollback y Limpieza (Opcional)

1. **Crear el script `cleanup_lab62.sh`:**

    ```bash
    sudo nano /opt/backup_scripts/cleanup_lab62.sh
    ```

    ```bash
    #!/bin/bash
    # Limpieza total del laboratorio (rollback)

    echo "⚠️  ADVERTENCIA: Este script eliminará todos los datos del laboratorio 6.2."
    read -p "¿Está seguro? Escriba 'SI' para continuar: " CONFIRMA

    if [ "$CONFIRMA" != "SI" ]; then
        echo "Cancelado."
        exit 0
    fi

    # Eliminar backups
    sudo rm -rf /var/backups/data_center/*
    sudo rm -rf /var/backups/files/*

    # Eliminar logs archivados
    sudo rm -rf /var/log/nginx/archive/*

    # Eliminar scripts
    sudo rm -rf /opt/backup_scripts/*

    # Eliminar tareas cron
    sudo crontab -r

    # Eliminar base de datos de prueba
    sudo mysql -e "DROP DATABASE IF EXISTS lab62_db;"

    echo "[OK] Limpieza completada."
    ```

    > **Explicación:** Un script de rollback permite revertir el entorno de laboratorio sin afectar configuraciones previas (como el hardening del Lab 5.1).

## ⚙️ Sección 3: Práctica en Grupo (Red entre Pares)

El objetivo es que cada grupo de **2 estudiantes** configure un escenario de backup y recuperación cruzado entre sus VMs.

### 1. Roles y Asignación por Grupo

| Integrante | Rol | VM a Configurar | Requisitos de Red |
| - | - | - | - |
| **Integrante 1** | **Servidor de Backups** | VM Backup (scripts, cron, almacenamiento) | Adaptador en modo **Puente (Bridge)** |
| **Integrante 2** | **Servidor de Datos** | VM DB (MariaDB, archivos web, logs) | Adaptador en modo **Puente (Bridge)** |

### 2. Tareas del Integrante 1 (Servidor de Backups)

1. **Configurar la VM en modo Puente** con IP estática.

2. **Generar clave SSH** y compartir la clave pública con el Integrante 2.

3. **Crear el script de backup remoto** adaptado a la IP del compañero:

    ```bash
    #!/bin/bash
    DB_HOST="IP_DEL_COMPAÑERO"
    DB_NAME="lab62_db"
    SSH_USER="usuario"
    BACKUP_DIR="/var/backups/compañero"
    FILE_DATE=$(date +%Y%m%d_%H%M)

    mkdir -p "$BACKUP_DIR"
    ssh "$SSH_USER@$DB_HOST" "sudo mysqldump --defaults-file=/etc/.my.cnf $DB_NAME" | gzip > "$BACKUP_DIR/$DB_NAME-$FILE_DATE.sql.gz"
    ```

4. **Ejecutar el backup** y verificar que el archivo se creó en su VM.

5. **Configurar cron** para que el backup se ejecute cada hora durante el laboratorio (para demostración):

    ```bash
    0 * * * * /opt/backup_scripts/db_backup_remoto.sh
    ```

### 3. Tareas del Integrante 2 (Servidor de Datos)

1. **Configurar la VM en modo Puente** con IP estática.

2. **Instalar MariaDB** y crear la base de datos `lab62_db` con al menos 5 registros de prueba.

3. **Crear el archivo `/etc/.my.cnf`** con credenciales seguras para `mysqldump`.

4. **Configurar UFW** para permitir:
    - SSH desde la IP del compañero.
    - MySQL (3306) solo desde la IP del compañero (si es necesario para pruebas adicionales).

5. **Simular un desastre:** Borrar intencionalmente la base de datos `lab62_db`.

### 4. Prueba de Recuperación Cruzada

1. **El Integrante 1 copia el backup más reciente** a la VM del compañero (o la VM del compañero lo descarga):

    ```bash
    scp /var/backups/compañero/lab62_db-*.sql.gz usuario@IP_COMPAÑERO:/tmp/
    ```

2. **El Integrante 2 restaura la base de datos:**

    ```bash
    zcat /tmp/lab62_db-*.sql.gz | sudo mysql
    sudo mysql -e "SELECT * FROM lab62_db.productos;"
    ```

3. **Verificación conjunta:**
    - ¿Se recuperaron todos los registros?
    - ¿Cuánto tiempo tomó la restauración (RTO)?
    - ¿El backup era consistente?

4. **Documentar el incidente simulado:**

    ```markdown
    ## Simulación de Recuperación - Grupo X

    - **Fecha:** [Fecha]
    - **Rol:** Integrante 2 (DB Server)
    - **Incidente:** Pérdida total de base de datos lab62_db (simulado DROP DATABASE)
    - **Backup realizado por:** Integrante 1
    - **Tiempo de restauración (RTO):** [X segundos]
    - **Registros recuperados:** 5/5
    - **Lección aprendida:** [Reflexión]
    ```

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual (backup, rotación, cron, restauración): 35 pts
- Desarrollo de la práctica grupal (backup cruzado, simulación de desastre, recuperación): 35 pts
- Informe detallado con capturas de pantalla, scripts documentados y análisis de RTO: 30 pts
