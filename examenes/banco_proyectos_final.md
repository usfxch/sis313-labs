# Banco de Proyectos — Examen Final SIS313

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

---

## 📋 Condiciones Generales del Proyecto Final

- **Modalidad:** Grupal (2 a 4 integrantes).
- **Duración:** 4 sesiones de laboratorio (2 semanas).
- **Infraestructura:** Máquinas virtuales en VirtualBox (mínimo 3 VMs, recomendado 4–7 VMs según el proyecto).
- **Evaluación:** Defensa oral + documentación técnica + demostración funcional.
- **Requisito mínimo:** Cada proyecto debe integrar al menos **4 temas avanzados** de los vistos en los laboratorios del semestre.

### 🖥️ Despliegue en Centro de Datos de la Carrera (Opcional)

Como extensión del alcance, los grupos pueden optar por desplegar su infraestructura en el **Centro de Datos de la Carrera de Ingeniería de Sistemas**, replicando el modelo utilizado en el Laboratorio 4.1 (*Plataforma HA, Balanceo de Carga y Monitoreo*):

- El docente asignará a cada grupo una **subred `/29` exclusiva** y un **VLAN ID único** dentro del segmento `192.168.(100+N).0/29`.
- El grupo debe solicitar al docente la provisión de una **VM con IP pública** que actuará como **Proxy Inverso / Balanceador de Carga** y puerta de enlace hacia Internet para toda la subred del grupo.
- Desde esta VM pública, el grupo expondrá sus servicios (web, monitoreo, DNS, etc.) hacia el exterior, tal como se practicó en el laboratorio con las rutas `https://vlanXXX-app.rootcode.com.bo` y `https://vlanXXX-monitoring.rootcode.com.bo`.
- Las VMs internas del grupo (aplicaciones, bases de datos, servidores DNS, etc.) se desplegarán dentro de la subred asignada y comunicarán con el exterior a través del proxy/balanceador.

> **🎯 Incentivo:** Los grupos que logren desplegar y demostrar su proyecto funcionando en el Centro de Datos de la Carrera (con acceso real desde Internet a través de la VM pública) obtendrán **+20 puntos adicionales** sobre la nota final del proyecto.
>
> **Nota:** El despliegue en el Data Center es opcional. Los grupos que no opten por esta modalidad desarrollarán el proyecto completamente en VirtualBox con reenvío de puertos local, sin penalización alguna.

### Temas Avanzados de la Asignatura

| Código | Tema |
|--------|------|
| T1 | Administración avanzada de GNU/Linux (FHS, permisos, systemd, logs) |
| T2 | Almacenamiento RAID y tolerancia a fallos |
| T3 | Alta disponibilidad (Keepalived, failover, VIP) |
| T4 | Proxy inverso y balanceo de carga (NGINX) |
| T5 | Segmentación de red con VLANs y routing inter-VLAN |
| T6 | Firewall, NAT y políticas de acceso (UFW / iptables) |
| T7 | DNS primario con BIND9 (zonas directas e inversas) |
| T8 | Despliegue de aplicaciones (Node.js / PHP + PM2) |
| T9 | Bases de datos centralizadas (MariaDB) |
| T10 | Monitoreo integral (Prometheus + Grafana) |
| T11 | Hardening integral (SSH, kernel, UFW, ofuscación) |
| T12 | Seguridad TLS/SSL (certificados, HSTS, cifrados) |
| T13 | Detección de intrusiones y respuesta a incidentes (Fail2ban) |
| T14 | Automatización con Bash (scripts, menús, health checks) |
| T15 | Backups automatizados, rotación y recuperación (tar, mysqldump, cron) |

---

## 🏦 Proyecto 1: Centro de Datos Miniatura para E-Commerce

**Descripción:**
Diseñar e implementar una infraestructura completa para una tienda en línea ficticia. El sistema debe soportar picos de tráfico, garantizar la disponibilidad continua de la base de datos de productos y permitir transacciones seguras. Se debe simular un ataque de fuerza bruta y demostrar la capacidad de recuperación ante desastres.

**Stack / Temas involucrados:** T2 (RAID 10 para BD), T3 (HA con Keepalived), T4 (NGINX balanceador), T9 (MariaDB), T10 (Prometheus + Grafana), T12 (TLS), T13 (Fail2ban), T15 (backups automatizados).

**Entregables:**
- Arquitectura de red documentada con diagrama.
- Demostración de failover al caer el nodo maestro.
- Reporte de monitoreo en Grafana con métricas de las 3 capas.
- Simulación de ataque y bloqueo vía Fail2ban.
- Restauración completa de la BD desde backup en menos de 5 minutos (RTO documentado).

---

## 🎓 Proyecto 2: Plataforma de Streaming Educativo USFX

**Descripción:**
Implementar una plataforma de video-lecciones para la universidad, donde los estudiantes accedan por nombre de dominio. La arquitectura debe incluir un balanceador que distribuya las peticiones a servidores de aplicación, una base de datos de usuarios, monitoreo en tiempo real y backups diarios de contenido académico. Todo el tráfico debe ser cifrado.

**Stack / Temas involucrados:** T4 (proxy inverso + balanceo NGINX), T7 (DNS BIND9 interno), T8 (Node.js + PM2), T9 (MariaDB), T10 (Grafana), T12 (TLS obligatorio), T14 (script de health check), T15 (backup de archivos y BD).

**Entregables:**
- Resolución DNS de `streaming.usfx.local` funcionando.
- Balanceo de carga entre al menos 2 servidores de aplicación.
- Dashboard de monitoreo con métricas de CPU, RAM y peticiones HTTP.
- Script Bash de health check que alerte si un nodo cae.
- Certificado TLS autofirmado y redirección forzada HTTP→HTTPS.

---

## 🏛️ Proyecto 3: Infraestructura Bancaria Simulada

**Descripción:**
Construir una red bancaria segmentada donde la VLAN de cajeros no pueda acceder a la VLAN de servidores core, pero sí a la VLAN de consultas. El servidor web debe estar en DMZ, la base de datos en una red interna crítica y todo el tráfico debe estar auditado. Se debe simular un incidente de seguridad y documentar la respuesta.

**Stack / Temas involucrados:** T5 (VLANs por departamento), T6 (UFW / iptables con políticas estrictas), T11 (hardening SSH y kernel), T12 (TLS), T13 (Fail2ban + análisis de logs), T14 (script de auditoría), T15 (backup cifrado de transacciones).

**Entregables:**
- Tabla de segmentación VLAN con políticas de acceso documentadas.
- Pruebas de conectividad: qué VLAN puede acceder a qué recurso.
- Reporte de hardening aplicado (SSH en puerto alterno, solo claves, etc.).
- Simulación de ataque Hydra y respuesta con Fail2ban.
- Documento de incidente con las 5 fases del ciclo de respuesta.

---

## 🚀 Proyecto 4: Despliegue de Microservicios con Automatización

**Descripción:**
Crear una infraestructura donde cada servicio (usuarios, pagos, notificaciones) corra en instancias Node.js gestionadas por PM2, detrás de un proxy inverso NGINX. Todo el despliegue de nuevas instancias, creación de usuarios del sistema y verificación de salud debe estar automatizado mediante scripts Bash. Los datos persistentes residen en MariaDB con RAID 1.

**Stack / Temas involucrados:** T1 (administración de servicios y permisos), T2 (RAID 1 para persistencia), T4 (NGINX proxy inverso + balanceo least_conn), T8 (Node.js + PM2), T9 (MariaDB), T14 (scripts de deploy, health check y menú interactivo), T15 (backup de BD y archivos).

**Entregables:**
- Script `deploy.sh` que instale Node.js, PM2 y despliegue una app desde Git.
- Script `health_check.sh` que verifique el estado de NGINX, PM2 y MariaDB.
- Menú interactivo de administración accesible por SSH.
- Demostración de caída de una app y redirección automática del balanceador.
- Backup remoto de la base de datos vía SSH con verificación de integridad.

---

## 🌐 Proyecto 5: Red Empresarial Multi-Sucursal

**Descripción:**
Simular una empresa con sede central y 2 sucursales. Cada sucursal tiene su propia VLAN (Ventas, Almacén, TI). El router Linux central debe enrutar el tráfico inter-VLAN, proveer acceso a Internet mediante NAT y filtrar conexiones. Cada sucursal resuelve nombres mediante un DNS local replicado. El servidor web de la intranet está en la DMZ.

**Stack / Temas involucrados:** T5 (VLANs y trunking), T6 (NAT, iptables, UFW), T7 (BIND9 con zona directa e inversa), T4 (NGINX para intranet), T11 (hardening básico), T14 (script de inventario de red).

**Entregables:**
- Diagrama de red con 3+ VLANs y sus subredes.
- Configuración de router Linux con enrutamiento inter-VLAN funcional.
- DNS que resuelva `intranet.empresa.local` y registros inversos.
- Firewall que impida que Ventas acceda a TI, pero sí a la DMZ.
- Script de inventario que haga ping y verifique puertos abiertos en cada VLAN.

---

## 🔒 Proyecto 6: SOC (Security Operations Center) Simulado

**Descripción:**
Implementar un centro de operaciones de seguridad donde un servidor "defensor" aloje servicios web y SSH protegidos por Fail2ban, mientras una máquina "atacante" simule escaneos y fuerza bruta. Todos los logs de autenticación y acceso web deben ser analizados mediante scripts Bash que generen alertas. El sistema debe incluir monitoreo visual de los intentos de intrusión.

**Stack / Temas involucrados:** T10 (Prometheus + Grafana para visualizar eventos), T11 (hardening SSH), T13 (Fail2ban, Hydra, Nmap), T14 (scripts de análisis de logs y alertas), T1 (gestión de logs con journalctl y awk).

**Entregables:**
- Dashboard en Grafana que muestre conexiones SSH activas y bloqueos.
- Script `analyze_threats.sh` que identifique IPs sospechosas y genere un reporte.
- Prueba de ataque de fuerza bruta documentada con capturas de logs.
- Política de firewall que bloquee rangos de IPs tras un umbral de intentos.
- Informe de incidente con identificación, contención, erradicación y recuperación.

---

## 🏥 Proyecto 7: Plataforma de Telemedicina Segura

**Descripción:**
Desplegar una infraestructura para una clínica que requiera alta disponibilidad, cifrado extremo-a-extremo y segmentación de red estricta. Los expedientes médicos residen en MariaDB sobre un arreglo RAID 5. La aplicación web solo es accesible por HTTPS con TLS 1.3. La red de equipos médicos está aislada en una VLAN sin salida a Internet.

**Stack / Temas involucrados:** T2 (RAID 5), T3 (HA con VIP), T5 (VLANs: médicos, admin, equipos), T6 (UFW estricto), T9 (MariaDB), T12 (TLS 1.2/1.3 + HSTS), T13 (Fail2ban), T15 (backup diario de expedientes).

**Entregables:**
- Arquitectura con al menos 3 VLANs y políticas documentadas.
- Prueba de acceso denegado desde VLAN de equipos médicos a Internet.
- Certificado TLS configurado con cifrados fuertes y cabeceras de seguridad.
- Fail2ban protegiendo SSH y logs de autenticación analizados.
- Simulación de restauración de expedientes tras pérdida de datos.

---

## ☁️ Proyecto 8: Cloud Privado con Automatización Total

**Descripción:**
Construir un mini-cloud privado donde la creación de usuarios, despliegue de servicios web, limpieza de logs y verificación de salud estén 100% automatizados mediante scripts Bash. El administrador interactúa con un menú interactivo. Los servicios desplegados incluyen NGINX, MariaDB y Node.js. Los backups se ejecutan sin intervención humana.

**Stack / Temas involucrados:** T1 (usuarios, permisos, systemd), T4 (NGINX), T8 (Node.js + PM2), T9 (MariaDB), T14 (scripts de deploy, menú interactivo, health check, limpieza de logs), T15 (backups con cron, rotación, restauración).

**Entregables:**
- Menú interactivo con al menos 5 opciones funcionales.
- Script que cree usuarios masivamente desde CSV en múltiples servidores vía SSH.
- Script de despliegue desatendido de la pila LEMP/Node.
- Health check automático que reinicie servicios caídos.
- Backups programados con cron y reporte de estado.

---

## 🎮 Proyecto 9: Servidor de Juegos Multijugador con Alta Disponibilidad

**Descripción:**
Implementar la infraestructura backend para un juego multijugador online. Se requieren servidores de aplicación redundantes balanceados, una base de datos central para puntuaciones, monitoreo en tiempo real de jugadores conectados y protección contra escaneos de puertos. Los datos de partidas se respaldan cada hora.

**Stack / Temas involucrados:** T3 (Keepalived + VIP), T4 (NGINX balanceo least_conn / IP hash), T8 (Node.js + PM2), T9 (MariaDB), T10 (Grafana con métricas de jugadores), T13 (Fail2ban), T15 (backup horario de partidas).

**Entregables:**
- Failover demostrado al detener el nodo maestro de Keepalived.
- Balanceo con persistencia de sesión (IP hash) para mantener jugadores en el mismo servidor.
- Dashboard de Grafana mostrando conexiones activas y uso de recursos.
- Simulación de escaneo Nmap y bloqueo por Fail2ban.
- Restauración de la tabla de puntuaciones desde backup.

---

## 🏭 Proyecto 10: Fábrica Inteligente 4.0 (OT/IT Convergente)

**Descripción:**
Simular una planta industrial donde la red de operaciones tecnológicas (OT: sensores, PLCs) está completamente aislada de la red de TI (servidores, oficina). Un router Linux gestiona ambas redes. Los datos de producción se almacenan en una BD protegida por RAID. El acceso a la zona OT está restringido mediante firewall y solo permite consultas de solo lectura desde TI.

**Stack / Temas involucrados:** T2 (RAID 1 para datos de producción), T5 (VLANs separadas OT/IT/DMZ), T6 (UFW / iptables estricto), T9 (MariaDB), T10 (monitoreo de recursos), T11 (hardening de acceso), T14 (script de health check industrial).

**Entregables:**
- Diagrama de red con aislamiento OT/IT.
- Prueba de ping fallido desde OT a Internet (aislamiento confirmado).
- Base de datos de producción accesible solo desde TI y solo por puerto 3306.
- RAID funcional con simulación de fallo de disco y reconstrucción.
- Script que reporte cada 5 minutos el estado de los servicios críticos.

---

## 📰 Proyecto 11: Portal de Gobierno Electrónico

**Descripción:**
Desplegar un portal de trámites municipales donde los ciudadanos (simulados) accedan por un dominio propio. La infraestructura incluye DNS autoritativo, balanceo de carga entre servidores web, base de datos de trámites, monitoreo de disponibilidad y respaldo automatizado de documentos. Todo acceso administrativo debe estar endurecido.

**Stack / Temas involucrados:** T4 (proxy + balanceo NGINX), T7 (BIND9 con zona directa e inversa), T9 (MariaDB), T10 (Grafana), T11 (hardening SSH), T12 (TLS), T15 (backup de archivos y BD).

**Entregables:**
- Dominio local funcional (`tramites.gob.local`) con resolución inversa.
- Balanceador distribuyendo tráfico entre 2+ servidores web.
- Panel de monitoreo mostrando uptime del portal.
- Acceso SSH solo por clave pública en puerto no estándar.
- Restauración completa del portal en una VM de contingencia.

---

## 🔄 Proyecto 12: Disaster Recovery Active-Passive

**Descripción:**
Diseñar una infraestructura con dos sitios: un centro de datos activo y un sitio de recuperación pasivo. El sitio activo ejecuta la aplicación con HA local; el sitio pasivo recibe réplicas de la base de datos y archivos mediante backups automatizados. Se debe simular un desastre completo en el sitio activo y demostrar la recuperación en el pasivo con un RTO medido.

**Stack / Temas involucrados:** T2 (RAID en ambos sitios), T3 (HA local con Keepalived), T9 (MariaDB), T14 (scripts de replicación y failover manual), T15 (backups remotos, restauración, RTO).

**Entregables:**
- Arquitectura active-passive documentada.
- Backup remoto automatizado de BD y archivos web cada 30 minutos.
- Simulación de desastre (apagado forzado del sitio activo).
- Recuperación funcional en el sitio pasivo con acceso web verificado.
- Documento con RTO y RPO calculados empíricamente.

---

## 🖥️ Proyecto 13: Proveedor de Hosting Web Compartido

**Descripción:**
Simular una empresa de hosting que aloje múltiples sitios web de clientes en un mismo servidor (o cluster). Cada cliente tiene su propio directorio, usuario del sistema y permisos restringidos. Un proxy inverso NGINX enruta por nombre de dominio. La creación de cuentas, la rotación de logs y los backups están automatizados.

**Stack / Temas involucrados:** T1 (usuarios, grupos, permisos, cuotas), T4 (NGINX proxy inverso + virtual hosts), T7 (DNS local), T11 (hardening y aislamiento), T14 (script de creación de cuentas y menú), T15 (backup por cliente, rotación de logs).

**Entregables:**
- Script `create_client.sh` que cree usuario, directorio web y configura virtual host.
- Al menos 3 sitios web accesibles por nombre de dominio distinto.
- Logs de NGINX rotados y archivados automáticamente.
- Backup individual por cliente con restauración demostrada.
- Política de firewall que aisle los directorios de los clientes.

---

## 🎓 Proyecto 14: Red Académica Campus Universitario

**Descripción:**
Diseñar la red de la universidad con VLANs por facultad (Ingeniería, Medicina, Derecho), una VLAN de servidores (DMZ) y una VLAN de administración TI. El router central enruta entre todas, pero solo TI tiene acceso total. Los servidores DNS, web y de archivos residen en la DMZ. Se debe monitorear el tráfico entre facultades.

**Stack / Temas involucrados:** T5 (4+ VLANs), T6 (UFW con políticas granulares), T7 (DNS primario para dominio `usfx.campus`), T4 (NGINX para portal institucional), T10 (monitoreo de tráfico inter-VLAN), T14 (script de inventario de equipos).

**Entregables:**
- Tabla de routing y políticas de acceso documentada.
- DNS que resuelva nombres de servidores y facultades.
- Prueba de aislamiento: Ingeniería no puede acceder a servidores admin.
- Monitoreo del router mostrando tráfico por interfaz VLAN.
- Script que genere un inventario de IPs activas por facultad.

---

## 🗳️ Proyecto 15: Sistema de Votación Electrónica con Auditoría

**Descripción:**
Implementar una plataforma de votación segura donde cada voto se registra en MariaDB y los logs de auditoría sean inmutables. La red está segmentada: votantes en una VLAN, servidores en otra y observadores en una tercera. Todo tráfico es HTTPS. Se debe detectar cualquier intento de acceso no autorizado y generar un trail de auditoría completo.

**Stack / Temas involucrados:** T5 (VLANs: votantes, servidores, observadores), T6 (firewall estricto), T9 (MariaDB con bind-address restringido), T11 (hardening completo), T12 (TLS + HSTS), T13 (Fail2ban + análisis de logs), T14 (script de auditoría), T15 (backup cifrado de votos).

**Entregables:**
- Arquitectura de red con segmentación y justificación.
- Base de datos de votos con restricción de acceso IP.
- Logs de auditoría analizados y reporte de intentos fallidos.
- Certificado TLS con verificación de rechazo a TLS 1.0/1.1.
- Script que genere un reporte de integridad de la votación.

---

## 🛒 Proyecto 16: Marketplace Local con Monitoreo y Escalabilidad

**Descripción:**
Desarrollar la infraestructura para un marketplace de comerciantes locales. La aplicación Node.js corre en múltiples instancias tras un balanceador NGINX. Los productos y pedidos se almacenan en MariaDB. Un sistema de monitoreo alerta cuando el uso de CPU supere el 80%. Los pedidos se respaldan cada hora y los archivos de productos cada día.

**Stack / Temas involucrados:** T4 (NGINX balanceo least_conn), T8 (Node.js + PM2 multi-instancia), T9 (MariaDB), T10 (Prometheus + Grafana con alertas), T12 (TLS), T14 (script de health check y menú), T15 (backups diferenciados).

**Entregables:**
- 2+ instancias de la app corriendo bajo PM2 con balanceo funcional.
- Dashboard de Grafana con alerta visual por alto consumo de CPU.
- Script de health check que reinicie instancias caídas.
- Backup automatizado de pedidos y archivos con cron.
- Restauración demostrada de la base de datos de pedidos.

---

## 📡 Proyecto 17: Infraestructura de Noticias con CDN Simulado

**Descripción:**
Simular el backend de un portal de noticias donde un proxy inverso NGINX distribuya peticiones estáticas (imágenes, HTML) a servidores cache locales y peticiones dinámicas a servidores de aplicación. El DNS interno resuelve múltiples subdominios (`www`, `static`, `api`). La infraestructura incluye monitoreo, backups de contenido editorial y protección contra escaneos.

**Stack / Temas involucrados:** T4 (NGINX proxy inverso + upstreams múltiples), T5 (VLANs: editorial, publico, admin), T7 (BIND9 con subdominios), T10 (monitoreo de servidores cache), T13 (Fail2ban), T15 (backup de contenido editorial).

**Entregables:**
- DNS funcional con `static.noticias.local`, `api.noticias.local`, etc.
- NGINX enrutando peticiones estáticas a un servidor y dinámicas a otro.
- Servidores cache respondiendo con contenido estático.
- Fail2ban bloqueando escaneos de rutas administrativas.
- Backup y restauración de artículos del portal.

---

## 🏗️ Proyecto 18: Plataforma de Gestión Documental Empresarial

**Descripción:**
Construir una infraestructura para gestionar documentos corporativos con alta disponibilidad. El almacenamiento de documentos reside en un RAID 10 compartido. Los empleados acceden por una aplicación web balanceada. La red está segmentada por departamentos y solo RRHH puede acceder a ciertos documentos. Todo está protegido por TLS y los accesos se auditan.

**Stack / Temas involucrados:** T2 (RAID 10 para documentos), T4 (NGINX balanceador), T5 (VLANs por departamento), T6 (UFW con restricciones de acceso a documentos), T9 (MariaDB de metadatos), T12 (TLS), T13 (auditoría de accesos), T15 (backup incremental de documentos).

**Entregables:**
- RAID 10 montado y verificado con datos de prueba.
- Políticas de firewall que restrinjan acceso a documentos por VLAN.
- Aplicación web accesible solo por HTTPS.
- Logs de acceso analizados para detectar consultas no autorizadas.
- Backup automatizado del RAID y la base de metadatos.

---

## 💬 Proyecto 19: Servicio de Mensajería con Persistencia y Seguridad

**Descripción:**
Implementar un servicio de mensajería interno tipo chat empresarial usando Node.js y MariaDB. El servicio debe ser de alta disponibilidad (Keepalived + 2 nodos de app) y todo el tráfico debe pasar por un proxy inverso con balanceo de carga. Los mensajes se respaldan cada 15 minutos y el sistema debe resistir un ataque de fuerza bruta.

**Stack / Temas involucrados:** T3 (Keepalived VIP), T4 (NGINX balanceo), T8 (Node.js + PM2), T9 (MariaDB), T10 (Grafana), T12 (TLS), T13 (Fail2ban), T15 (backup frecuente de mensajes).

**Entregables:**
- VIP funcional que migre al nodo backup en caso de fallo.
- Balanceo distribuyendo conexiones WebSocket/chat entre nodos.
- Dashboard de monitoreo mostrando mensajes por minuto y recursos.
- Simulación de ataque y bloqueo con Fail2ban.
- Restauración de la conversación desde backup con verificación de integridad.

---

## 🧪 Proyecto 20: Centro de Cómputo de Exámenes en Línea

**Descripción:**
Diseñar la infraestructura para un sistema de exámenes en línea con miles de estudiantes concurrentes simulados. Se requiere balanceo de carga, una base de datos central para calificaciones, segmentación de red (estudiantes, docentes, servidores), monitoreo de la carga del sistema y backups en tiempo real de las calificaciones. El acceso está restringido y auditado.

**Stack / Temas involucrados:** T4 (NGINX balanceo round robin / least_conn), T5 (VLANs: estudiantes, docentes, servidores), T6 (UFW), T9 (MariaDB), T10 (Prometheus + Grafana), T11 (hardening SSH), T13 (Fail2ban), T15 (backup continuo de calificaciones).

**Entregables:**
- Arquitectura de red con 3+ VLANs y tabla de accesos.
- Balanceador soportando carga simulada con Apache Bench (`ab`).
- Dashboard de monitoreo mostrando capacidad del sistema bajo estrés.
- Firewall restringiendo acceso de estudiantes solo a puerto 80/443.
- Backup de calificaciones restaurado y verificado.

---

## 🧬 Proyecto 21: Laboratorio de Bioinformática con Almacenamiento Masivo

**Descripción:**
Simular un laboratorio de análisis genómico donde los datasets masivos se almacenan en un RAID 5 con tolerancia a fallos. Los científicos acceden a los datos mediante una aplicación web balanceada. La red está segmentada: área de secuenciación, área de análisis y área administrativa. Los datasets se respaldan de forma remota y los logs de acceso se rotan automáticamente.

**Stack / Temas involucrados:** T2 (RAID 5 para datasets), T4 (NGINX proxy + balanceo), T5 (VLANs de laboratorio), T6 (firewall inter-VLAN), T9 (MariaDB de metadatos), T14 (script de rotación de logs y health check), T15 (backup remoto de datasets, verificación de integridad).

**Entregables:**
- RAID 5 funcional con simulación de fallo y reconstrucción documentada.
- Proxy inverso balanceando peticiones a la app de análisis.
- Script de rotación de logs de acceso a datasets.
- Backup remoto de archivos genómicos con verificación de checksum.
- Health check automatizado del RAID y la base de datos.

---

## 🚚 Proyecto 22: Logística y Transporte con Rastreo en Tiempo Real

**Descripción:**
Implementar una infraestructura para una empresa de logística que rastree flotas de vehículos en tiempo real. Los datos GPS se reciben en servidores Node.js balanceados, se almacenan en MariaDB y se visualizan en dashboards de monitoreo. La red de oficinas está segmentada de la red de operaciones de campo. Todo el tráfico administrativo usa SSH endurecido y los datos se respaldan cada 6 horas.

**Stack / Temas involucrados:** T3 (Keepalived para alta disponibilidad de recepción), T4 (NGINX balanceo), T5 (VLANs: oficina, operaciones, DMZ), T8 (Node.js + PM2), T9 (MariaDB), T10 (Grafana con métricas de flota), T11 (hardening SSH), T15 (backups programados de rutas).

**Entregables:**
- Arquitectura HA con VIP para recepción de datos.
- Balanceador distribuyendo carga entre 2+ nodos de procesamiento.
- Dashboard mostrando eventos por segundo y latencia.
- SSH endurecido: puerto alterno, solo claves, sin root.
- Backup y restauración de la base de datos de rutas documentada.

---

## ✅ Rúbrica General de Evaluación (100 pts + 20 pts bonus)

| Criterio | Puntos |
|----------|--------|
| **Diseño de arquitectura** (diagrama, elección de tecnologías, justificación) | 15 |
| **Implementación técnica** (servicios funcionando, red operativa, sin errores críticos) | 25 |
| **Integración de temas** (mínimo 4 temas avanzados demostrados) | 15 |
| **Seguridad y hardening** (firewall, TLS/SSL, SSH, segmentación) | 15 |
| **Monitoreo / Automatización** (Grafana, scripts, menús, cron) | 10 |
| **Respaldo y recuperación** (backup funcional, restauración demostrada, RTO) | 10 |
| **Documentación y defensa** (informe técnico, capturas, presentación oral) | 10 |
| **Despliegue en Data Center** (opcional, VM pública + subred asignada + acceso real desde Internet) | **+20** |

---

**Nota para los estudiantes:**
Los proyectos pueden adaptarse al contexto real del grupo (ejemplo: cambiar "fábrica" por "cooperativa", "banco" por "cooperativa de ahorro"), siempre y cuando se mantengan los requisitos técnicos y la complejidad equivalente. Consulte con el docente antes de realizar modificaciones sustanciales.
