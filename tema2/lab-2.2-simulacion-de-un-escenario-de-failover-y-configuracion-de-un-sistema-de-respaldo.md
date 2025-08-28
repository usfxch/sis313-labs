# Laboratorio 2.2: Simulación de un escenario de Failover y configuración de un sistema de Respaldo

## 🎯 Objetivo del Laboratorio

- **Configurar un entorno** para simular un escenario de failover (conmutación por error) usando herramientas de software.

- **Demostrar la alta disponibilidad** de un servicio al verificar su continuidad durante la simulación de una falla.

- **Configurar un sistema de respaldo** básico y restaurar datos, para diferenciar la alta disponibilidad del respaldo de datos.

- **Comprender la importancia** de los planes de contingencia para la continuidad operacional.

## 🛠️ Sección 1: Preparación del Entorno Virtual

1. **Creación de Máquinas Virtuales:** Crea dos máquinas virtuales con Ubuntu Server 24.04 LTS en VirtualBox. Nómbralas `Lab2.2-servidor-ha1` y `Lab2.2-servidor-ha2`.

2. **Configuración de Red:** Asegúrate de que ambas máquinas estén en la misma red y puedan comunicarse entre sí.

3. **Instalación de Software:** Instala un servidor web ligero (como **Nginx** o **Apache2**) en ambas máquinas virtuales. Para Nginx, usa el siguiente comando:
    ```bash
    sudo apt update && sudo apt install nginx -y
    ```

4. **Instalación de Keepalived:** Instala `keepalived` en ambas máquinas:
    ```bash
    sudo apt install keepalived -y
    ```

## 💻 Sección 2: Práctica guiada

En esta sección, se te guiará a través de ejercicios y ejemplos para que te familiarices con los conceptos de failover y respaldo. Al final, debes poder explicar los **Puntos Clave** de la práctica.

### Ejercicio 1: Configuración de Alta Disponibilidad con Keepalived

- **Concepto:** Se utiliza la herramienta **Keepalived** para crear un clúster de alta disponibilidad. Asigna una IP flotante o virtual (`VIP`, por sus siglas en inglés) que se moverá automáticamente entre los dos servidores en caso de falla.

- **Pasos a seguir:**
    1. **Configuración de Keepalived en `Lab2.2-servidor-ha1` (MAESTRO):**
        - Edita el archivo de configuración:
            ```bash
            sudo nano /etc/keepalived/keepalived.conf
            ```

        - Reemplaza el contenido del archivo con la siguiente configuración, adaptando la IP flotante (ej. `192.168.1.100`) y las interfaces de red (ej. `enp0s3`) según tu subred y tu entorno:
            ```keepalived
            vrrp_instance VI_1 {
                state MASTER                    # Estado inicial del nodo
                interface enp0s3                # Interfaz de red a utilizar (ej. eth0, enp0s3)
                virtual_router_id 51            # ID único para el grupo de clústeres (mismo en ambos nodos)
                priority 100                    # Prioridad (más alta para el nodo principal)
                advert_int 1                    # Intervalo de anuncio (segundos)
                authentication {                # Autenticación para que los nodos se comuniquen de forma segura
                    auth_type PASS
                    auth_pass secret            # Clave secreta para la autenticación
                }
                virtual_ipaddress {
                    192.168.1.100/24 dev enp0s3 # La IP virtual compartida (VIP) y la interfaz de red
                }
            }
            ```
    2. **Configuración de Keepalived en `Lab2.2-servidor-ha2` (ESCLAVO):**
        - **Edita el archivo de configuración:**
            ```bash
            sudo nano /etc/keepalived/keepalived.conf
            ```
        
        - Reemplaza el contenido del archivo con la siguiente configuración, adaptando la IP y la interfaz. Lo más importante es cambiar `state MASTER` a `state BACKUP` y `priority` a un valor menor (ej. `100`).
            ```keepalived
            vrrp_instance VI_1 {
                state BACKUP                    # Estado inicial del nodo
                interface enp0s3                # Misma interfaz que el principal
                virtual_router_id 51            # Mismo ID que el principal
                priority 50                     # Prioridad (más baja para el nodo de respaldo)
                advert_int 1
                authentication {
                    auth_type PASS
                    auth_pass clave_secreta     # Misma clave secreta que el principal
                }
                virtual_ipaddress {
                    192.168.1.100/24 dev enp0s3 # Misma IP virtual que el principal
                }
            }
            ```

        - **Inicia los servicios:** En ambas máquinas, reinicia el servicio de Keepalived:
            ```bash
            sudo systemctl restart keepalived
            ```