# 📄 Plantilla 2: Formato de Diapositivas para Presentación Final

Esta plantilla sugiere la estructura y el contenido para las diapositivas de la defensa final del proyecto, asegurando que se cubran los aspectos teóricos y prácticos de la asignatura.

## Diapositiva 1: Título y Presentación

- **Título Principal:** Proyecto Final SIS313: [Título del Proyecto]

- **Subtítulo:** Implementando la [Plataforma/Solución]

- **Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

- **Integrantes:** Nombres y Apellidos (Con rol)

- **Semestre:** 2/2025

## Diapositiva 2: 🎯 Objetivo y Problema Resuelto

- **Objetivo:** El objetivo puntual del proyecto (ej., Optimización del Rendimiento del LMS Moodle mediante Sistemas de Caché ).

- **Problema / Justificación:** ¿Qué falla o riesgo operacional se aborda? (Ej. Evitar caídas del sistema de inscripción por saturación de tráfico ).

- **Solución Propuesta:** Breve resumen de la solución (ej. Implementación de una capa de Virtual Queue con Rate Limiting).

## Diapositiva 3: 🛠️ Tecnologías Empleadas

- **Tecnologías Principales:** (Logos y nombres)

    - **Servidores/OS:** Linux [Distribución]

    - **Capa de Aplicación:** [Nginx/Apache, Nextcloud/GitLab, etc.]

    - **Base de Datos:** [MariaDB/PostgreSQL/MongoDB]

    - **HA/Automatización:** [Keepalived, Ansible, PM2, Cron]

## Diapositiva 4: 🧠 Temas de la Asignatura Puestos en Práctica (SIS313)

- **Tópicos Avanzados:**

    - **Alta Disponibilidad y Tolerancia a Fallos (T2):** (Ej. Replicación Maestro-Esclavo, Failover VRRP).

    - **Seguridad y Hardening (T5):** (Ej. WAF ModSecurity, Hardening SSH, Cifrados TLS).

    - **Automatización y DRP (T6):** (Ej. Playbooks Ansible, Plan de Backups Incremental).

    - **Optimización y Escalabilidad (T4):** (Ej. Caché Redis/Memcached, Balanceo de Carga L7).

## Diapositiva 5: 🌐 Diseño de la Infraestructura (Esquemático)

- **Diagrama de Topología:** Debe ser claro y mostrar los flujos de tráfico.

    - Identificar las **Capas** (Proxy, Aplicación, DB) y la **Segmentación de Red** (Subnets, VLANs).

    - Marcar claramente los **Puntos de Redundancia (HA)** (ej. IP Virtual / VIP).

## Diapositiva 6: ⚙️ Estrategia de Implementación

- **Estrategia:** Describir la metodología de despliegue.

    - *Si se usó Ansible:* Despliegue con CI/CD (Automatización T6).

    - *Si es un clúster de DB:* Estrategia de separación de Lectura/Escritura.

    - **Demostración Práctica:** Mostrar un comando o un fragmento de código crucial que demuestre un concepto avanzado. (Ej. El output de un `ansible-playbook` o la configuración de `keepalived.conf`).

## Diapositiva 7: ✔️ Validación y Conclusiones

- **Pruebas Clave:** Mostrar los resultados de las pruebas críticas.

    - *Prueba de Failover:* (Ej. Muestre la caída del MASTER y el servicio sigue funcionando).

    - *Prueba de Monitoreo:* (Ej. Un dashboard de Grafana con la carga de CPU o el hit rate de la caché).

- **Conclusión:** ¿Se cumplió el objetivo? ¿Qué aprendizaje técnico fue el más valioso?