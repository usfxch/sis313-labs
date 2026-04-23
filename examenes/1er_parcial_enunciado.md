# 🧪 EXAMEN PRÁCTICO - PRIMER PARCIAL SIS313
## Infraestructura, Plataformas Tecnológicas y Redes (1/2026)

**Duración Total:** 90 minutos (30 minutos por ejercicio) 

**Herramientas Necesarias:** Máquina Virtual Ubuntu Server

**Formato:** 3 ejercicios prácticos con enunciados a desarrollar

---

## 📋 INSTRUCCIONES GENERALES

1. **Lee cuidadosamente** cada enunciado antes de empezar
2. **Desarrolla la solución** usando lo que aprendiste (NO es copiar-pegar)
3. **Toma capturas de pantalla** de los resultados importantes
4. **Documenta los comandos** que utilizaste
5. **Verifica el resultado** antes de pasar al siguiente paso
6. **Algunos pasos incluyen comandos como referencia**, úsalos solo si es necesario
7. **Tiempo:** Tienes 30 minutos por ejercicio

---

## ✅ RÚBRICA DE EVALUACIÓN

| Criterio | Puntos | Descripción |
|----------|--------|-------------|
| **Cumplimiento del enunciado** | 40% | ¿Se completó todo lo solicitado? |
| **Comandos correctos** | 30% | ¿Los comandos funcionan correctamente? |
| **Documentación** | 20% | ¿Hay capturas y explicaciones? |
| **Limpieza y organización** | 10% | ¿El trabajo está ordenado y limpio? |

---

# 🔧 EJERCICIO 1: Administración Integral de Sistema Operativo
## Basado en Laboratorio 1.1 y 1.2 (30 minutos)

### 📌 ENUNCIADO

Tu organización requiere preparar un servidor de desarrollo que será compartido por un equipo de trabajo. Eres responsable de:

1. **Gestionar acceso de usuarios:** El servidor debe tener dos usuarios que no sean root: uno llamado `developer` (con permisos sudo) y otro llamado `guest` (sin permisos administrativos). Ambos necesitan un *home* configurado correctamente.

2. **Diagnosticar el estado del sistema:** Debes verificar el uso actual de CPU, memoria RAM, espacio en disco y determinar si hay recursos críticos que estén al límite. El servidor será usado para compilar código, por lo que necesita al menos 1.5 GB de RAM disponible.

3. **Configurar almacenamiento seguro:** Los datos sensibles de la empresa deben almacenarse en una ubicación separada con permisos restrictivos (acceso solo para `developer` y lectura para grupo `developers`). Debes crear estas estructuras.

4. **Auditar seguridad:** Revisa los intentos de login fallidos en los últimos 24 horas usando logs del sistema y documenta cualquier actividad sospechosa.

5. **Validar servicios críticos:** Verifica que SSH está corriendo en el servidor y documenta cómo verificarías las conexiones activas.

### 🎯 TAREAS ESPECÍFICAS

1. **Crear y configurar usuarios:**
   - Crear usuario `developer` con contraseña y añadirlo al grupo `sudo`
   - Crear usuario `guest` con contraseña pero sin permisos administrativos
   - Verificar correctamente los directorios home para ambos

2. **Diagnóstico de recursos del sistema:**
   - Ejecutar `top`, `free -h`, `df -h` y documentar los resultados
   - Usar `htop` o `vmstat` para analizar la distribucion de recursos
   - Calcular y reportar el porcentaje de RAM disponible respecto al total
   - Identificar qué proceso está consumiendo más memoria

3. **Configuración de almacenamiento seguro:**
   - Crear directorio `/opt/datos_empresa` con permisos 750 (explica qué significan estos permisos)
   - Cambiar propietario a `developer:developers`
   - Crear archivo de test dentro y verificar permisos desde ambos usuarios
   - Documentar si `guest` puede leer vs escribir

4. **Auditoría de seguridad:**
   - Revisar archivo `/var/log/auth.log` para intentos fallidos en últimas 24h
   - Ejecutar `grep "Failed password" /var/log/auth.log` (o similar)
   - Contar cuántos intentos fallidos hubo y desde qué usuarias
   - Documentar en un informe breve

5. **Validación de servicios:**
   - Verificar estado de SSH: `sudo systemctl status ssh`
   - Documentar quién está conectado actualmente (comando `w` o `who`)
   - Probar conexión SSH desde otro user local si es posible

**Capturas requeridas:**
  - Estado de usuarios creados (`cat /etc/passwd` con filtro a tus usuarios)
  - Output de `free -h` y `df -h` con análisis escrito
  - Estructura de permisos en `/opt/datos_empresa`
  - Resultado de auditoría de auth.log (mínimo 5 líneas relevantes)
  - Estado de servicio SSH
  - Intento de acceso a datos_empresa desde usuario `guest` (mostrando denegación)

**Comandos esperados:**
  ```bash
  sudo useradd -m -s /bin/bash -G sudo developer
  sudo useradd -m -s /bin/bash guest
  echo "developer:PASSWORD" | sudo chpasswd
  echo "guest:PASSWORD" | sudo chpasswd
  free -h && df -h
  ps aux --sort=-%mem | head -5
  sudo mkdir -p /opt/datos_empresa
  sudo chown developer:developers /opt/datos_empresa
  sudo chmod 750 /opt/datos_empresa
  grep "Failed password" /var/log/auth.log | wc -l
  sudo systemctl status ssh
  sudo journalctl -u ssh -n 10
  ```

## 📝 ENTREGA DEL EJERCICIO 1

Debe incluir:
  - **Comandos utilizados** y **capturas de pantalla** de los comandos ejecutados con una pequeña descripción

---

# 🐳 EJERCICIO 2: Implementación y Recuperación de RAID
## Basado en Laboratorios 2.1 (30 minutos)

### 📌 ENUNCIADO

Una empresa de desarrollo necesita implementar un sistema de almacenamiento redundante para su servidor de aplicaciones. Se requiere:

1. **Crear un arreglo RAID 1 (Mirror)** con dos discos virtuales para datos críticos. Este arreglo debe alojar los datos de la base de datos de producción.

2. **Verificar la integridad del RAID:** Después de crear el arreglo, debes documentar su estado, confirmar que está sincronizado y verificar que ambos discos participan en el espejo.

3. **Simular una falla de disco:** Debes simular la falla de uno de los discos del arreglo RAID 1 y documentar cómo el sistema responde. El RAID debe seguir operativo con un disco funcionando.

4. **Recuperación y re-sincronización:** Después de marcar un disco como fallido, debes retirar el disco de forma segura y luego realizar la recuperación del arreglo.

5. **Análisis de rendimiento y tolerancia:** Documentar la diferencia de rendimiento entre:
   - Lectura/escritura en un disco individual
   - Lectura/escritura en el arreglo RAID 1 (con ambos discos)
   - Lectura/escritura en el arreglo RAID 1 degradado (con un disco fallido)

### 🎯 TAREAS ESPECÍFICAS

1. **Preparación y creación de RAID:**
   - Identificar los discos disponibles con `sudo fdisk -l`
   - Crear arreglo RAID 1 con al menos 2 discos de 1 GB cada uno
   - Crear sistema de archivos ext3 en el RAID
   - Montar en `/mnt/raid-datos`
   - Verificar sincronización con `cat /proc/mdstat`

2. **Verificación de estado del RAID:**
   - Ejecutar `sudo mdadm --detail /dev/md0` y documentar información:
     - Versión del arreglo
     - Nivel del RAID
     - Número de dispositivos activos y totales
     - Estado de sincronización
   - Verificar que ambos discos estén en estado "clean"
   - Documentar el UUID del arreglo

3. **Simulación de falla (Marcar disco como fallido):**
   - Sin desconectar físicamente, marcar uno de los discos como fallido
   - Capturar el estado con `cat /proc/mdstat` (debe mostrar "recovering" o "degraded")
   - Verificar que el arreglo sigue operativo y accesible: crear un archivo de prueba en el RAID
   - Documentar el tiempo que toma el RAID degradado en responder

4. **Recuperación del arreglo:**
   - Retirar el disco fallido
   - Documentar el estado después de la remoción
   - Para recuperación total: reemplazar con nuevo disco
   - Monitorear la resincronización con `watch cat /proc/mdstat`
   - Documentar cuánto tiempo toma la resincronización

**Capturas requeridas:**
  - Resultado de `sudo fdisk -l` mostrando los discos disponibles
  - Estado inicial del RAID con `cat /proc/mdstat` (sincronizado)
  - Salida de `sudo mdadm --detail /dev/md0` (información completa)
  - Estado de archivos en `/proc/mdstat` cuando el disco está marcado como fallido
  - Intento de acceso a archivos en RAID con un disco fallido (demostrando operabilidad)
  - Estado final después de remoción del disco
  - Progreso de resincronización durante recuperación

**Comandos esperados:**
  ```bash
  sudo fdisk -l
  sudo mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sdb /dev/sdc
  sudo mkfs.ext4 /dev/md0
  sudo mkdir -p /mnt/raid-datos
  sudo mount /dev/md0 /mnt/raid-datos
  cat /proc/mdstat
  sudo mdadm --detail /dev/md0
  sudo mdadm /dev/md0 --fail /dev/sdc
  sudo mdadm /dev/md0 --remove /dev/sdc
  sudo mdadm /dev/md0 --add /dev/sdc
  watch cat /proc/mdstat
  df -h /mnt/raid-datos
  time dd if=/dev/zero of=/mnt/raid-datos/test.img bs=1M count=100
  ```

## 📝 ENTREGA DEL EJERCICIO 2

Debe incluir:
  - Describir cada paso del proceso e incluir **capturas** mostrando los estados del RAID en cada fase
  - **Análisis detallado** de:
    - Por qué RAID 1 es importante para datos críticos
    - Cómo el arreglo mantiene disponibilidad con un disco fallido

---

# 🚀 EJERCICIO 3: Configuración de Alta Disponibilidad con Failover
## Basado en Laboratorio 2.2 (30 minutos)

### 📌 ENUNCIADO

Una aplicación web crítica de una empresa necesita estar disponible 24/7 sin interrupciones. Se requiere implementar un clúster de alta disponibilidad usando Keepalived que:

1. **Configure dos servidores Nginx** (servidor primario y servidor de respaldo) en máquinas virtuales distintas.

2. **Implemente una IP virtual flotante (VIP)** mediante Keepalived que se mueva automáticamente entre servidores ante cualquier falla.

3. **Demuestre el failover automático** simulando la caída del servidor primario y verificando que:
   - La aplicación sigue siendo accesible a través de la VIP
   - El tráfico se redirecciona al servidor de respaldo automáticamente
   - No hay interrupción del servicio (o es imperceptible)

4. **Recuperación automática:** Cuando el servidor primario se recupera, el servicio debe volver a conmutarse de forma ordenada.

5. **Monitoreo y análisis:** Documentar el comportamiento del clúster, tiempos de failover y disponibilidad del servicio.

### 🎯 TAREAS ESPECÍFICAS

1. **Preparación de máquinas virtuales:**
   - Crear/clonar dos máquinas Ubuntu Server 24.04 LTS
   - Configurar red en modo bridge para ambas
   - Asegurar conectividad entre ellas: `ping <IP otra maquina>`
   - Documentar las IPs asignadas a ambos servidores (ej: 192.168.1.100 y 192.168.1.101)
   - Seleccionar una tercera IP para la VIP (ej: 192.168.1.150)

2. **Instalación de servicios:**
   - En ambos servidores:
     ```bash
     sudo apt update && sudo apt install nginx keepalived -y
     ```
   - Verificar que Nginx inicia correctamente en ambos
   - Crear páginas web identificables:
     ```bash
     echo "Servidor MASTER (o BACKUP)" | sudo tee /var/www/html/index.html
     ```
   - Probar acceso local con `curl localhost:80`

3. **Configuración de Keepalived (SERVIDOR PRIMARIO/MASTER):**
   - Editar `/etc/keepalived/keepalived.conf`
   - Configurar como MASTER con prioridad 101
   - Configurar VIP y autenticación
   - Iniciar servicio: `sudo systemctl restart keepalived`
   - Verificar estado: `sudo systemctl status keepalived`
   - Verificar IP virtual con: `ip addr show` (debe aparecer la VIP)

4. **Configuración de Keepalived (SERVIDOR DE RESPALDO/BACKUP):**
   - Editar `/etc/keepalived/keepalived.conf`
   - Configurar como BACKUP con prioridad 100 (menor que MASTER)
   - Usar MISMA autenticación que MASTER
   - Usar MISMA VIP que MASTER
   - Iniciar servicio
   - Verificar que la IP virtual NO aparece en este servidor (por ahora)

5. **Simulación de failover:**
   - **Fase 1 (Normal):** 
     - Desde máquina cliente, acceder a `curl http://<VIP>` 
     - Documentar que responde el MASTER
     - Verificar con `sudo systemctl status keepalived` en ambos servidores
   
   - **Fase 2 (Simular falla del MASTER):**
     - Detener Nginx en MASTER: `sudo systemctl stop nginx`
     - O detener Keepalived en MASTER: `sudo systemctl stop keepalived`
     - ESPERAR 3-5 segundos (intervalo de detección)
     - Verificar que la VIP migró al BACKUP: `ip addr show` en BACKUP
   
   - **Fase 3 (Verificar failover):**
     - Acceder a VIP desde cliente: `curl http://<VIP>`
     - Documentar que ahora responde el BACKUP
     - Verificar logs de Keepalived: `sudo journalctl -u keepalived -n 20`
   
   - **Fase 4 (Recuperación):**
     - Restaurar servicio en MASTER: `sudo systemctl start nginx`
     - O restaurar: `sudo systemctl start keepalived`
     - Verificar logs para documentar la reconexión
     - Confirmar que VIP regresó al MASTER (o está disponible desde ambos)

6. **Análisis de tiempos y disponibilidad:**
   - Documentar el tiempo exacto de detección de falla
   - Documentar el tiempo de conmutación de la VIP
   - Estimar el tiempo de indisponibilidad (downtime)

**Capturas requeridas:**
  - Configuración de red de ambos servidores (ifconfig o `ip addr`)
  - Captura del archivo de configuración `/etc/keepalived/keepalived.conf` de ambos servidores
  - Estado de Keepalived en ambos servidores antes del failover
  - Estado de IP virtual (VIP) en MASTER antes del failover
  - Acceso a aplicación mediante VIP funcionando normalmente
  - Momento exacto cuando se detiene servicio en MASTER
  - Estado de IP virtual en BACKUP durante failover (mostrando la VIP)
  - Acceso a aplicación mediante VIP después de failover (demostrando continuidad)
  - Logs de Keepalived mostrando eventos de failover
  - Estado final cuando MASTER se recupera

**Comandos esperados:**
  ```bash
  # Diagnostico inicial
  ip addr show
  ping -c 3 <IP otra maquina>
  sudo systemctl status nginx
  sudo systemctl status keepalived
  
  # Configuración
  sudo nano /etc/keepalived/keepalived.conf
  sudo systemctl restart keepalived
  
  # Pruebas de conectividad
  curl http://localhost:80
  curl http://<VIP>:80
  
  # Simulación de falla
  sudo systemctl stop nginx  # o stop keepalived
  sudo systemctl start nginx
  
  # Monitoreo
  watch -n 1 'ip addr show | grep <VIP>'
  sudo journalctl -u keepalived -f
  sudo journalctl -u keepalived -n 50 --no-pager
  ```

## 📝 ENTREGA DEL EJERCICIO 3

Debe incluir:
  - **Capturas** documentando cada fase:
    - Configuración inicial
    - Estado normal del clúster
    - Simulación de falla
    - Failover en progreso
    - Recuperación del servidor principal
  - **Estimación de tiempos**:
    - Tiempo de detección de falla
    - Tiempo de conmutación de VIP
    - Tiempo total de indisponibilidad
  - **Análisis crítico:**
    - Ventajas de alta disponibilidad vs respaldo simple
    - Diferencia entre failover y backup
    - Escenarios donde esta solución es apropiada
    - Limitaciones y mejoras posibles
    - Cómo se comportaría con 3 o más servidores

---

*Examen Práctico SIS313 - Primer Semestre 2026*
*Duración: 90 minutos*
*Puntuación total: 90 puntos (~30 por ejercicio)*
*Formato: Enunciados a desarrollar (NO copiar-pegar)*
