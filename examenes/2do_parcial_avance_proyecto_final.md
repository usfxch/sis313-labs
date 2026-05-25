# Enunciado del Segundo Parcial — Evaluación por Avance del Proyecto Final

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026


**Fecha de evaluación:** 29 de mayo de 2026

**Entrega de documentos en eCampus:** 29 de mayo de 2026 hasta las 09:00 (requisito obligatorio)

**Sorteo de horarios:** 29 de mayo de 2026 a las 10:00

**Inicio de evaluaciones:** 29 de mayo de 2026 a las 11:00

**Tiempo de presentación por grupo:** 10 minutos (estricto)

**Puntaje total:** 100 puntos

---

## 📋 Instrucciones Generales

1. Cada grupo debe llegar el día del parcial con su avance **completamente preparado y ensayado**.
2. La evaluación se realiza en formato **presentación-demostración** de 10 minutos por grupo.
3. Los grupos deben traer sus laptops con las VMs encendidas y listas antes de ingresar a la sala de evaluación.
4. Se permite el uso de apuntes personales y laboratorios previos como referencia.
5. Queda **estrictamente prohibido** el uso de inteligencia artificial generativa (ChatGPT, Copilot, etc.) durante la presentación.
6. **Entrega previa obligatoria en eCampus (hasta 29/05 a las 09:00):**
   - **Tabla de infraestructura** actualizada (1 página).
   - **Bitácora de avance** con mínimo 3 entradas (1 página máximo).
   - **Diagrama de arquitectura** con leyenda de estado (operativo / en configuración / pendiente).
   > ⚠️ **Los grupos que no hayan subido estos 3 documentos a eCampus antes de las 09:00 no podrán acceder a la defensa y serán calificados con 0 (cero) puntos en el parcial.**
7. El **sorteo de horarios** se realizará a las 10:00 del día de la evaluación. Los grupos deben estar atentos y presentarse 5 minutos antes de su horario asignado.
8. Al ingresar a presentar, cada grupo debe traer sus laptops con las VMs **ya encendidas** y listas para demostrar.
9. La calificación se compone de: funcionalidad del avance (60%), documentación entregada (20%) y defensa grupal (20%).

---

## 🎯 Enfoque de la Evaluación

El segundo parcial mide el **avance técnico real** del proyecto final mediante una demostración en vivo. No hay tiempo de configurar nada durante los 15 minutos; el grupo solo **muestra lo que ya funciona** y responde preguntas puntuales del docente.

---

## ⏱️ Distribución Recomendada de los 10 Minutos

Para que el grupo aproveche el tiempo y el docente pueda calificar ordenadamente, se sugiere la siguiente estructura:

| Fase | Duración | Actividad |
|------|----------|-----------|
| **Apertura** | 30 seg | Identificación del grupo, número de proyecto y escenario. |
| **Hito 1** | 1 min | Muestra rápida de VMs encendidas, IPs y `ping` cruzado. |
| **Hito 2** | 4 min | Demostración de los 2 servicios core (pruebas funcionales rápidas). |
| **Hito 3** | 1 min 30 seg | Verificación de SSH endurecido, firewall activo y permisos. |
| **Hito 4** | 2 min | Explicación breve del diagrama y bitácora; defensa individual de 15 segundos por persona. |
| **Cierre** | 1 min | Preguntas puntuales del docente y resumen final. |

> **⚠️ Advertencia:** El docente detendrá la presentación a los 10 minutos exactos. Ensayen en casa con un cronómetro.

---

## 🏗️ Hitos de Avance Obligatorios

El puntaje de cada hito se asigna **solo si se demuestra en vivo** durante los 15 minutos.

---

### Hito 1: Infraestructura Base y Conectividad (20 puntos)

**Tiempo sugerido:** 1 minuto

**Objetivo:** La plataforma física/lógica está montada y operativa.

#### Requisitos mínimos

1. **Máquinas virtuales levantadas:** Al menos el 60 % de las VMs definidas en la arquitectura del proyecto deben estar creadas, encendidas y con acceso por SSH.
2. **Configuración de red funcional:**
   - Interfaces de red configuradas según el diseño del grupo.
   - Conectividad básica verificada con `ping` entre VMs esenciales.
3. **Asignación de IPs estáticas:** Las VMs críticas tienen IP estática configurada.
4. **Tabla de infraestructura entregada** al docente al inicio, con:
   - Nombre de cada VM, rol, SO, IP y estado actual.

#### Evidencias rápidas (mostrar en pantalla)

- `ip addr` en la VM principal.
- `ping` cruzado entre 2 VMs.
- `systemctl status ssh` (o el servicio SSH configurado).

---

### Hito 2: Servicios Core en Ejecución (35 puntos)

**Tiempo sugerido:** 4 minutos

**Objetivo:** Al menos **dos temas avanzados** del proyecto deben estar implementados, integrados y funcionando.

#### Requisitos mínimos

El grupo demuestra **dos o más** de estos temas ya operativos, según su proyecto:

| Opción | Tema | Evidencia mínima (rápida) |
|--------|------|---------------------------|
| A | **DNS (T7)** | `dig @<ip-dns> www.dominio.local` desde otra VM mostrando respuesta. |
| B | **Proxy Inverso / Balanceo (T4)** | `curl` al balanceador mostrando respuesta de diferentes backends; o `nginx -t` + navegación. |
| C | **Alta Disponibilidad (T3)** | `ip addr` en nodo master mostrando VIP; simular caída (`sudo systemctl stop keepalived`) y mostrar migración. |
| D | **Base de Datos Centralizada (T9)** | `mysql -h <ip-bd> -u ... -e "SHOW DATABASES;"` desde una VM remota. |
| E | **Aplicación Desplegada (T8)** | Navegador o `curl` mostrando la aplicación respondiendo por IP o dominio interno. |
| F | **Segmentación de Red / VLANs (T5)** | `ping` entre 2 VLANs diferentes pasando por el router Linux. |
| G | **RAID y Almacenamiento (T2)** | `cat /proc/mdstat` mostrando estado `active`; `df -h` mostrando el RAID montado. |

> **Nota:** Si el proyecto no incluye alguna opción, el docente validará temáticas equivalentes previa coordinación.

#### Regla de oro para este hito

- **No basta con "instalado".** El servicio debe responder a una prueba real en ese momento.
- Si un servicio está caído o mal configurado, el grupo puede optar por **no mostrarlo** y perder esos puntos, pero no se permite reconfigurar en vivo.

---

### Hito 3: Seguridad y Hardening Inicial (25 puntos)

**Tiempo sugerido:** 1 minuto 30 segundos

**Objetivo:** El proyecto no está desprotegido. Hay una capa mínima de seguridad aplicada.

#### Requisitos mínimos

1. **Hardening de SSH (10 pts):**
   - Puerto no estándar **O** acceso por clave pública **O** `PermitRootLogin no`.
   - Servicio SSH activo y demostrable.

2. **Firewall activo (10 pts):**
   - UFW o iptables habilitado con política de denegar por defecto.
   - Solo puertos estrictamente necesarios abiertos.
   - Mostrar `ufw status verbose` o `iptables -L -v -n`.

3. **Usuarios y permisos (5 pts):**
   - Existencia de usuarios diferenciados (no todo como root).
   - Permisos restrictivos en al menos 2 directorios críticos.

#### Evidencias rápidas

- `grep -E "^(Port|PermitRootLogin)" /etc/ssh/sshd_config`
- `ufw status verbose`
- `ls -la /var/www/` o `/opt/scripts/`

---

### Hito 4: Planificación y Defensa Grupal (20 puntos)

**Tiempo sugerido:** 2 minutos 30 segundos totales (incluye preguntas del docente)

**Objetivo:** El grupo demuestra organización, comprensión del proyecto y reparto de tareas.

#### Requisitos mínimos

1. **Diagrama de arquitectura con leyenda de estado (5 pts):**
   - Refleja el estado *actual*.
   - Usa colores o etiquetas: verde (operativo), amarillo (en configuración), gris (pendiente).

2. **Bitácora de avance (5 pts):**
   - Mínimo 3 entradas con fecha, actividad, responsable y dificultad superada.
   - Entregada al inicio; no se lee en voz alta completa.

3. **Defensa grupal e individual breve (10 pts):**
   - Cada integrante dispone de **15 segundos** para decir:
     - *"Yo construí/configuré X"*
     - *"Me falta por hacer Y para la siguiente entrega"*
   - El docente puede hacer **1 pregunta puntual** sobre comandos, IPs o decisiones de diseño.

---

## 📊 Rúbrica de Evaluación

| Hito | Criterio | Puntos | Puntos obtenidos |
|------|----------|--------|------------------|
| **Hito 1** | Infraestructura base y conectividad | 20 | |
| | VMs levantadas (≥60 %) y accesibles | 8 | |
| | Red configurada y ping funcional | 7 | |
| | Tabla de infraestructura entregada | 5 | |
| **Hito 2** | Servicios core en ejecución | 35 | |
| | Tema avanzado 1 operativo y verificado | 17 | |
| | Tema avanzado 2 operativo y verificado | 17 | |
| | Integración coherente entre ambos servicios | 1 | |
| **Hito 3** | Seguridad y hardening inicial | 25 | |
| | SSH endurecido | 10 | |
| | Firewall activo y restrictivo | 10 | |
| | Usuarios y permisos diferenciados | 5 | |
| **Hito 4** | Planificación y defensa | 20 | |
| | Diagrama actualizado con leyenda de estado | 5 | |
| | Bitácora de avance entregada (3+ entradas) | 5 | |
| | Defensa grupal clara y reparto de tareas evidente | 10 | |
| | **Total** | **100** | |

---

## ✅ Checklist de Llegada (para el grupo)

Antes de su horario de presentación, verifique:

- [ ] Todas las VMs están **encendidas** y los servicios críticos ya están activos.
- [ ] Se ha ensayado en casa con un **cronómetro de 10 minutos**.
- [ ] Se tiene lista una **terminal abierta** por VM para no perder tiempo en conectarse.
- [ ] Se ha preparado una **carpeta o escritorio** con las capturas de pantalla por si el docente las solicita.
- [ ] Se ha impreso (o se tiene en PDF listo para enviar) la **tabla de infraestructura**, **bitácora** y **diagrama**.
- [ ] Cada integrante sabe exactamente qué decir en sus 20 segundos de defensa.

---

**¡Éxitos en la preparación y en la presentación!**
