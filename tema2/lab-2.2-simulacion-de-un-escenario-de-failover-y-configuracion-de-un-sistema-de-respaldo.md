# Laboratorio 2.2: Simulación de un escenario de Failover y configuración de un sistema de Respaldo

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

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

En esta sección, se te guiará a través de ejercicios y ejemplos para que te familiarices con los conceptos de failover y respaldo. 

**IMPORTANTE:** Para estos ejercicios se considera como ejemplo el uso de las IPs `172.16.30.11` para el servidor `Lab2.2-servidor-ha1` (MAESTRO), `172.16.30.12` para el servidor `Lab2.2-servidor-ha2` (ESCLAVO) y `172.16.30.100` para la IP virtual compartida del clúster. En tu caso, debes reemplazar estas IPs por las que te asigne tu red.

### Ejercicio 1: Configuración de Alta Disponibilidad con Keepalived

- **Concepto:** Se utiliza la herramienta **Keepalived** para crear un clúster de alta disponibilidad. Asigna una IP flotante o virtual (`VIP`, por sus siglas en inglés) que se moverá automáticamente entre los dos servidores en caso de falla.

- **Pasos a seguir:**
    1. **Configuración de Keepalived en `Lab2.2-servidor-ha1` (MAESTRO):**
        - Edita el archivo de configuración:
            ```bash
            sudo nano /etc/keepalived/keepalived.conf
            ```

        - Reemplaza el contenido del archivo con la siguiente configuración, adaptando la IP flotante (ej. `172.16.30.100`) y las interfaces de red (ej. `enp0s3`) según tu subred y tu entorno:
            ```keepalived
            vrrp_instance VI_1 {
                state MASTER                # Estado inicial del nodo
                interface enp0s3            # Interfaz de red a utilizar (ej. eth0, enp0s3)
                virtual_router_id 51        # ID único para el grupo de clústeres (mismo en ambos nodos)
                priority 101                # Prioridad (más alta para el nodo principal)
                advert_int 1                # Intervalo de anuncio (segundos)
                authentication {            # Autenticación para que los nodos se comuniquen de forma segura
                    auth_type PASS
                    auth_pass secret        # Clave secreta para la autenticación
                }
                virtual_ipaddress {
                    172.16.30.100           # La IP virtual compartida (VIP) y la interfaz de red
                }
            }
            ```
    2. **Configuración de Keepalived en `Lab2.2-servidor-ha2` (ESCLAVO):**
        - Edita el archivo de configuración:
            ```bash
            sudo nano /etc/keepalived/keepalived.conf
            ```
        
        - Reemplaza el contenido del archivo con la siguiente configuración, adaptando la IP y la interfaz. Lo más importante es cambiar `state MASTER` a `state BACKUP` y `priority` a un valor menor (ej. `100`).
            ```keepalived
            vrrp_instance VI_1 {
                state BACKUP                # Estado inicial del nodo
                interface enp0s3            # Misma interfaz que el principal
                virtual_router_id 51        # Mismo ID que el principal
                priority 100                # Prioridad (más baja para el nodo de respaldo)
                advert_int 1
                authentication {
                    auth_type PASS
                    auth_pass secret        # Misma clave secreta que el principal
                }
                virtual_ipaddress {
                    172.16.30.100           # Misma IP virtual que el principal
                }
            }
            ```

        - **Inicia los servicios:** En ambas máquinas, reinicia el servicio de Keepalived:
            ```bash
            sudo systemctl restart keepalived
            ```

    3. **Simulación de un escenario de Failover:**
        - Verifica que el clúster esté funcionando ingresando a `http://172.16.30.100` desde un navegador.
            - Puedes modificar el archivo `/var/www/html/index.nginx-debian.html` para identificar el servidor que está respondiendo.
                ```bash
                sudo nano /var/www/html/index.nginx-debian.html
                ```
            
            - También puedes identificar si la IP virtual (o flotante) esté habilitada.
                ```bash
                ip a show enp0s3
                ```

        - Apaga el servidor `Lab2.2-servidor-ha1` (MAESTRO):
            ```bash
            sudo poweroff
            ```
        
        - Verifica si aún puedes ingresar a `http://172.16.30.100`.
    
    4. **Configuración del script de monitoreo:** Crea un script que verifique si el servicio web está activo.

        - Crea el archivo del script en ambas máquinas:
            ```bash
            sudo nano /etc/keepalived/check_nginx.sh
            ```

        - Añade el siguiente contenido: 
            ```bash
            #!/bin/bash
            systemctl status nginx > /dev/null 2>&1
            if [ $? -eq 0 ]; then
                exit 0
            else
                exit 1
            fi
            ```
        
         - Asigna permisos de ejecución al archivo:
            ```bash
            sudo chmod 755 /etc/keepalived/check_nginx.sh
            ```

        - Modificar el contenido del archivo `/etc/keepalived/keepalived.conf` del servidor `Lab2.2-servidor-ha1` (MAESTRO):
            ```keepalived
            vrrp_script check_nginx {
                script "/etc/keepalived/check_nginx.sh" # Ruta al script de verificación
                interval 2                              # Comprobar cada 2 segundos
                weight 20                               # Ponderación a la prioridad si el script es exitoso
            }
            vrrp_instance VI_1 {
                state MASTER                # Estado inicial del nodo
                interface enp0s3            # Interfaz de red a utilizar (ej. eth0, enp0s3)
                virtual_router_id 51        # ID único para el grupo de clústeres (mismo en ambos nodos)
                priority 101                # Prioridad (más alta para el nodo principal)
                advert_int 1                # Intervalo de anuncio (segundos)
                authentication {            # Autenticación para que los nodos se comuniquen de forma segura
                    auth_type PASS
                    auth_pass secret        # Clave secreta para la autenticación
                }
                virtual_ipaddress {
                    172.16.30.100           # La IP virtual compartida (VIP) y la interfaz de red
                }
                track_script {
                    check_nginx             # Nombre del script
                }
            }
            ```

         - Modificar el contenido del archivo `/etc/keepalived/keepalived.conf` del servidor `Lab2.2-servidor-ha2` (ESCLAVO):
            ```keepalived
            vrrp_script check_nginx {
                script "/etc/keepalived/check_nginx.sh" # Ruta al script de verificación
                interval 2                              # Comprobar cada 2 segundos
                weight 20                               # Ponderación a la prioridad si el script es exitoso
            }
            vrrp_instance VI_1 {
                state BACKUP                # Estado inicial del nodo
                interface enp0s3            # Misma interfaz que el principal
                virtual_router_id 51        # Mismo ID que el principal
                priority 100                # Prioridad (más baja para el nodo de respaldo)
                advert_int 1
                authentication {
                    auth_type PASS
                    auth_pass secret        # Misma clave secreta que el principal
                }
                virtual_ipaddress {
                    172.16.30.100           # Misma IP virtual que el principal
                }
                track_script {
                    check_nginx             # Nombre del script
                }
            }
            ```

        - Detiene la ejecución el servicio de `nginx` en el servidor `Lab2.2-servidor-ha1` (MAESTRO):
            ```bash
            sudo systemctl stop nginx
            ```

        - Verifica si el servidor web continua funcionando en `http://172.16.30.100`.

### Ejercicio 2: Creación de un Respaldo de Datos con rsync y crontab

- **Concepto:** La alta disponibilidad protege la continuidad del servicio, pero no los datos. Un sistema de respaldo es necesario para protegerse contra la pérdida de datos. `rsync` es una herramienta poderosa que sincroniza archivos de forma incremental, lo que es muy eficiente. Además, usaremos `crontab` para automatizar el proceso.

- **Pasos a seguir:**
    1. **Instala** `rsync` en ambas máquinas:
        ```bash
        sudo apt install rsync -y
        ```

    2. **Creación de un directorio `data` con archivos importantes** en el servidor `Lab2.2-servidor-ha1` (MAESTRO):
        ```bash
        mkdir ~/data
        ```

        ```bash
        cd ~/data && touch importante{1..100}.dat
        ```

    3. **Configura la clave pública SSH para la conexión automática:**

        - En el servidor `Lab2.2-servidor-ha1` (MAESTRO), genera la clave SSH. No ingreses ninguna contraseña cuando se te solicite:
            ```bash
            ssh-keygen -t rsa
            ```

        - Copia la clave pública al servidor `Lab2.2-servidor-ha1` (ESCLAVO). Reemplaza usuario por el nombre de usuario y la IP de tu máquina esclava:
            ```bash
            ssh-copy-id <usuario>@<IP del servidor ESCLAVO>
            ```
        
        - Verifica la conexión automática:
            ```bash
            ssh <usuario>@<IP del servidor ESCLAVO>
            ```
            > Con esto, ya no debería pedirte una contraseña.

    4. **Sincronización el directorio** `~/data` del servidor `Lab2.2-servidor-ha1` (MAESTRO) en el servidor `Lab2.2-servidor-ha2` (ESCLAVO):
        ```bash
        # Reemplaza el usuario y la IP del servidor ESCLAVO
        rsync -azP ~/data <usuario>@<IP del servidor ESCLAVO>:~/data
        ```
    
    5. Modifica el contenido de un archivo del directorio `~/data` con `nano` del servidor MAESTRO y vuelve a sincronizar:
        ```bash
        # Reemplaza el nombre de usuario y la IP del servidor ESCLAVO
        rsync -azP ~/data <usuario>@<IP del servidor ESCLAVO>:~/data
        ```
        > Identifica qué tipo de sincronización se realizó. ¿Fue total o incremental?
    
    6. Automatización con `crontab`:

        - Abre el editor de crontab en servidor MAESTRO: 
            ```bash
            crontab -e
            ```
            > Selecciona un editor de texto (si te lo solicita).

        - Añade la siguiente línea al final del archivo para que el respaldo se ejecute cada minuto. Asegúrate de reemplazar `usuario` con tu nombre de usuario y el IP del servidor ESCLAVO.
            ```bash
            # Reemplaza el nombre de usuario y la IP del servidor ESCLAVO
            * * * * * rsync -azP ~/data <usuario>@<IP del servidor ESCLAVO>:~/data
            ```
            Explicación de los asteriscos de `crontab`:

            `*` (primer asterisco): Minuto (0-59)

            `*` (segundo asterisco): Hora (0-23)

            `*` (tercer asterisco): Día del mes (1-31)

            `*` (cuarto asterisco): Mes (1-12)

            `*` (quinto asterisco): Día de la semana (0-6, donde 0 es domingo)
    7. **Simulación de pérdida de datos:** Borra un archivo del servidor MAESTRO:
        ```bash
        rm importante10.dat
        ```

## ⚙️ Sección 3: Práctica en Grupo

Esta es tu oportunidad para demostrar que puedes aplicar los conceptos de manera colaborativa. Deberán coordinarse para configurar un clúster real entre sus computadoras.

### 🤝 Escenario de la Práctica

Trabajando en parejas, cada estudiante configurará una máquina virtual que actuará como un nodo en un clúster de alta disponibilidad.

- El **Estudiante A** configurará su máquina virtual como el nodo `MASTER` (`servidor-ha1`).

- El **Estudiante B** configurará su máquina virtual como el nodo `BACKUP` (`servidor-ha2`).

- Ambos estudiantes deben tener sus máquinas virtuales configuradas con un **adaptador de red en modo puente** para que puedan comunicarse entre sí en la red local del laboratorio.

- Asegúrense de que las direcciones IP de sus máquinas virtuales estén en el mismo rango de red que la red local.

### 🚀 Tareas a Realizar

1. **Configuración de Adaptador de Red:**

    - En VirtualBox, vayan a la configuración de su máquina virtual.

    - Naveguen a **Red > Adaptador 1**.

    - Seleccionen el menú desplegable y cambien **"NAT"** a **"Adaptador Puente"**.

    - Seleccionen la tarjeta de red de su computadora anfitriona que está conectada al internet.

2. **Validación de Conectividad:**

    - Asegúrense de que las máquinas de ambos estudiantes puedan hacer `ping` entre sí.

3. **Configuración del Clúster HA:**

    - Cada estudiante debe configurar su respectivo nodo (`MASTER` y `BACKUP`) siguiendo las instrucciones del Ejercicio 1 de la Sección 2.

    - Utilicen una **dirección IP flotante** que no esté en uso en la red local del laboratorio.

4. **Simulación de Failover (Conmutación por error):**

    - Desde la máquina del Estudiante A, verifiquen que el servicio web está funcionando a través de la IP flotante.

    - El Estudiante A detendrá el servicio web (`sudo systemctl stop nginx`).

    - El Estudiante B, desde su máquina, intentará acceder a la IP flotante y verificará que su servidor ha tomado el control.

5. **Simulación de Respaldo y Recuperación:**

    - El Estudiante A creará el archivo de datos y lo configurará para ser respaldado cada minuto en la máquina del Estudiante B usando `crontab`.

    - El Estudiante A simulará una pérdida de datos borrando algunos archivos.

    - El Estudiante B verificará la presencia de los archivos de datos en su máquina.

### ✅ Evaluación del Laboratorio

La evaluación se basará en un informe detallado con capturas de pantalla que demuestren cada uno de los pasos realizados. El informe debe incluir:

1. **Captura de la configuración de red en VirtualBox** mostrando el adaptador puente.

2. **Demostración de la conectividad** (ej. resultado del comando `ping`).

3. **Captura de pantalla de la configuración de Keepalived** en ambos nodos.

4. **Verificación del servicio** antes y después del failover.

5. **Demostración del respaldo** mostrando la entrada de crontab y la presencia del archivo en el servidor del Estudiante B.

6. **Conclusiones Conjuntas:** Un breve resumen de las lecciones aprendidas sobre cómo el trabajo en equipo, la **Alta Disponibilidad** y los **sistemas de respaldo** son necesarios para un plan de contingencia completo.