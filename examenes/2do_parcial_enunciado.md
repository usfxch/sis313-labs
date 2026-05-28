# Enunciado del Segundo Parcial

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

**Duración:** 2 horas 30 minutos

**Puntaje total:** 100 puntos

---

## 📋 Instrucciones Generales

1. El examen es **práctico** y se desarrolla en máquinas virtuales con **Ubuntu Server 24.04 LTS**.
2. Se utilizará **una única VM** para todo el examen. El estudiante debe configurarla y trabajar sobre ella durante los 3 ejercicios.
3. Se permite el uso de apuntes personales, diapositivas de clase y laboratorios previos.
4. Queda **estrictamente prohibido** el uso de inteligencia artificial generativa (ChatGPT, Copilot, etc.) durante el examen.
5. Al finalizar, el estudiante debe presentar:
   - Las **capturas de pantalla** solicitadas en cada ejercicio.
   - Los **scripts** desarrollados (si aplica).
   - Un **resumen breve** de los comandos ejecutados.
6. La evaluación considera: funcionalidad (70%), documentación/capturas (20%) y orden/presentación (10%).

---

## 🏢 Escenario: TechBol S.R.L.

La empresa **TechBol S.R.L.** está implementando su infraestructura interna de servicios. Se le ha asignado como administrador de sistemas para desplegar y asegurar los siguientes servicios en un único servidor:

- Un **servidor DNS primario** para resolver nombres internos.
- Un **servidor web seguro** (Nginx) accesible por nombre de dominio.
- **Políticas de seguridad** (firewall, SSH hardened, TLS).
- **Automatización** de monitoreo y backups.

El servidor utilizará la IP estática `192.168.100.10/24` y el dominio interno será `techbol.local`.

---

## Ejercicio 1: Servicios de Red — DNS y Servidor Web (35 puntos)

### Objetivo
Configurar un servidor DNS primario con BIND9 y un servidor web Nginx integrado al dominio.

### Requisitos

1. **Configuración de red estática** en la VM:
   - IP: `192.168.100.10/24`
   - Gateway: `192.168.100.1`
   - DNS: `127.0.0.1` (auto-resolución)

2. **Servidor DNS (BIND9):**
   - Instalar BIND9.
   - Configurar la zona directa para `techbol.local` en `/etc/bind/named.conf.local`.
   - Crear el archivo de zona `/etc/bind/db.techbol.local` con los siguientes registros:
     - **SOA** con `dns.techbol.local.` como servidor primario.
     - **NS** apuntando a `dns.techbol.local.`
     - **A** para `dns` → `192.168.100.10`
     - **A** para `www` → `192.168.100.10`
     - **CNAME** para `web` → `www.techbol.local.`
   - Configurar la **zona inversa** para el segmento `192.168.100.0/24` en `/etc/bind/named.conf.local`.
   - Crear el archivo de zona inversa `/etc/bind/db.192.168.100` con el registro **PTR** para la IP `192.168.100.10` apuntando a `dns.techbol.local.`.
   - Verificar la configuración con `named-checkconf`, `named-checkzone` (zona directa) y `named-checkzone` (zona inversa).
   - Reiniciar BIND9.

3. **Servidor Web (Nginx):**
   - Instalar Nginx.
   - Crear el directorio `/var/www/techbol.local` con un `index.html` que contenga:
     - El nombre del estudiante.
     - La fecha del examen.
     - El texto: `"Servidor Web de TechBol S.R.L. - DNS Integrado"`.
   - Configurar un Virtual Host en `/etc/nginx/sites-available/techbol.local` que responda a `www.techbol.local`.
   - Activar el sitio y reiniciar Nginx.

4. **Verificación:**
   - Desde la propia VM, ejecutar:
     ```bash
     dig @127.0.0.1 www.techbol.local
     dig -x 192.168.100.10 @127.0.0.1
     curl http://www.techbol.local
     ```
   - **Entregable:** Captura de pantalla del resultado de los tres comandos, mostrando resolución directa e inversa.

---

## Ejercicio 2: Seguridad y Hardening — Protección del Servidor (35 puntos)

### Objetivo
Aplicar medidas de hardening integral al servidor desplegado en el Ejercicio 1.

### Requisitos

1. **Hardening de SSH:**
   - Cambiar el puerto de SSH a `2222`.
   - Deshabilitar el acceso del usuario `root` (`PermitRootLogin no`).
   - Limitar a 3 intentos de autenticación (`MaxAuthTries 3`).
   - Reiniciar el servicio SSH y verificar que escucha en el puerto 2222.

2. **Fail2ban (Protección contra fuerza bruta):**
   - Instalar Fail2ban.
   - Crear `/etc/fail2ban/jail.local` configurando el jail `[sshd]` para:
     - Vigilar el puerto `2222`.
     - `maxretry = 3` intentos fallidos.
     - `findtime = 300` segundos (5 minutos).
     - `bantime = 600` segundos (10 minutos).
   - Reiniciar Fail2ban y verificar el estado con `fail2ban-client status sshd`.

3. **Firewall UFW:
   - Establecer política de **denegar por defecto** todo el tráfico entrante.
   - Permitir explícitamente:
     - Puerto `53/udp` (DNS)
     - Puerto `80/tcp` (HTTP)
     - Puerto `443/tcp` (HTTPS)
     - Puerto `2222/tcp` (SSH hardened)
   - Habilitar UFW y mostrar `ufw status verbose`.

3. **SSL/TLS Autofirmado en Nginx:**
   - Generar un certificado autofirmado con OpenSSL (vigencia 365 días, RSA 2048 bits).
   - Configurar Nginx para escuchar en HTTPS (puerto 443) con:
     - Protocolos TLS 1.2 y TLS 1.3 únicamente.
     - Cabeceras de seguridad obligatorias:
       - `Strict-Transport-Security` (HSTS)
       - `X-Frame-Options: SAMEORIGIN`
     - `server_tokens off;`
   - Redirigir el tráfico HTTP (80) → HTTPS (443).
   - Reiniciar Nginx.

4. **Verificación:**
   - Desde la propia VM, ejecutar:
     ```bash
     curl -k -I https://www.techbol.local
     openssl s_client -connect 192.168.100.10:443 -tls1_2 </dev/null
     sudo fail2ban-client status sshd
     ```
   - **Entregable:** Captura de pantalla que muestre las cabeceras de seguridad, la negociación TLS 1.2 y el estado de Fail2ban.

---

## Ejercicio 3: Automatización — Scripts y Backups (30 puntos)

### Objetivo
Crear scripts de automatización para el monitoreo del servidor y el backup de los archivos web.

### Requisitos

1. **Script de Health Check (`/opt/scripts/health_check.sh`):**
   - Debe verificar que el servicio **Nginx** esté activo.
   - Debe verificar que el servicio **Fail2ban** esté activo.
   - Debe verificar que el uso del disco raíz (`/`) no supere el **85%**.
   - Si alguna de las condiciones falla, debe registrar una alerta en `/var/log/techbol_health.log` con la fecha y el problema detectado.
   - Si todo está correcto, debe registrar un mensaje de estado normal indicando que los servicios críticos operan correctamente.
   - Otorgar permisos de ejecución y probarlo manualmente.

2. **Script de Backup (`/opt/scripts/backup_web.sh`):**
   - Debe crear un archivo comprimido `.tar.gz` del directorio `/var/www/techbol.local`.
   - El nombre del backup debe incluir la fecha y hora: `techbol-web-YYYYMMDD_HHMM.tar.gz`.
   - El backup debe almacenarse en `/var/backups/techbol/`.
   - Si el backup se crea correctamente, debe mostrar un mensaje de confirmación con el tamaño del archivo.
   - Otorgar permisos de ejecución y probarlo manualmente.

3. **Planificación con Cron:**
   - Configurar el `health_check.sh` para ejecutarse **cada 5 minutos**.
   - Configurar el `backup_web.sh` para ejecutarse **una vez por hora en punto** (ej. 10:00, 11:00, etc.).
   - Redirigir la salida estándar y errores de ambas tareas a `/dev/null`.
   - Verificar que las tareas están cargadas con `crontab -l`.

4. **Verificación:**
   - Mostrar el contenido de `/var/log/techbol_health.log` después de al menos una ejecución del health check.
   - Listar los archivos en `/var/backups/techbol/` para confirmar que el backup fue generado.
   - **Entregable:** Captura de pantalla del `crontab -l`, del log de health check y del directorio de backups.

---

## 📊 Criterios de Evaluación por Ejercicio

| Criterio | Ej. 1 (35 pts) | Ej. 2 (35 pts) | Ej. 3 (30 pts) |
|----------|----------------|----------------|----------------|
| Funcionalidad y configuración correcta | 25 pts | 25 pts | 20 pts |
| Capturas de pantalla / evidencias | 7 pts | 7 pts | 7 pts |
| Orden, presentación y explicación breve | 3 pts | 3 pts | 3 pts |

---

## ✅ Checklist Final de Entrega

Antes de concluir el examen, asegúrese de entregar:

- [ ] **Ejercicio 1:** Captura de `dig` y `curl` funcionando correctamente.
- [ ] **Ejercicio 2:** Captura de cabeceras HTTPS, verificación TLS 1.2 y estado de Fail2ban.
- [ ] **Ejercicio 3:** Captura de `crontab -l`, log de health check y directorio de backups.
- [ ] **Scripts:** Archivos `health_check.sh` y `backup_web.sh` en `/opt/scripts/`.
- [ ] **Resumen breve:** Lista de los comandos principales ejecutados (máximo 1 página).

---

**¡Éxitos en el examen!**
