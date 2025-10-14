# Laboratorio 4.1: Plataforma HA, Balanceo de Carga y Monitoreo

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

Implementar una arquitectura web de **Alta Disponibilidad (HA)** con balanceo de carga, segregación de servicios y monitoreo integral de rendimiento.

- Configurar un **Proxy Inverso (Nginx)** para balancear el tráfico entre dos servidores de aplicaciones.

- Desplegar dos instancias de aplicación (Node.js) gestionadas por **PM2** en modo de producción.

- Instalar y configurar una Base de Datos centralizada (**MariaDB**) y asegurar su acceso.

- Integrar un sistema de **Monitoreo (Prometheus + Grafana)** para medir la disponibilidad y el rendimiento de la plataforma.


## 🛠️ Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando 3 Máquinas Virtuales (VMs) con **Ubuntu Server 24.04 LTS**, simulando la segregación con interfaces de red y reenvío de puertos.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.10.0/29`.

    | VM | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - |
    | **VM 1** | **PROXY + MONITORING** | NAT (Internet) + Red Interna | `192.168.10.2`|
    | **VM 2** | **APLICACIONES (Apps 1 y 2)** | Red Interna | `192.168.10.3` |
    | **VM 3** | **BASE DE DATOS (DB)** | Red Interna | `192.168.10.4` |

    - **Gateway (GW)**: La interfaz interna de la VM 1 actuará como router/puerta de enlace para el tráfico de salida de las VMs 2 y 3. (Se configura a mano el *binding*).

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab4_1`.

    2. **Configurar Interfaces de las VMs:**

        - **VM 1 (Proxy/Monitoreo):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab4_1`).

        - **VM 2 (Aplicaciones):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_1`).

        - **VM 3 (Base de Datos):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_1`).
    
    3. **Reenvío de Puertos (Port Forwarding) en VM 1 (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM 1 **(Proxy/Monitoring)** para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP	| 127.0.0.1	| **2222** | 10.0.2.15 | 22 | Acceso Remoto |
        | **PROXY** | TCP | 127.0.0.1 | **80** | 10.0.2.15 | 80 | Acceso Web (Nginx) |
        | **GRAFANA** | TCP | 127.0.0.1 | **8080** | 10.0.2.15 | 3000 | Monitoreo Web |


## 💻 Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Proxy Inverso y Balanceo de Carga (VM 1)

1. **Configurar IPs Estáticas:** Asignar las IPs `192.168.10.2` (interna) a la VM 1 y las IPs correspondientes a las VMs 2 y 3 en Netplan.

2. **Instalar Nginx:**

    ```bash
    sudo apt install nginx -y
    ```

3. **Configurar el Balanceador:**

    - Configurar el servidor y crear el bloque `upstream` en `/etc/nginx/sites-available/default` con las IPs de la VM 2 (donde correrán las dos apps, en diferentes puertos).

        ```nginx
        upstream loadbalancer {
            server 192.168.10.3:3001; # App 1
            server 192.168.10.3:3002; # App 2
        }

        server {
            listen 80;
            server_name _;

            location / {
                proxy_pass http://loadbalancer;
            }
        }
        ```
4. **Habilitar UFW:** Abrir solo los puertos 22, 80 y 8080 (para Grafana) en la interfaz NAT. 

### Ejercicio 2: Servidores de Aplicaciones (VM 2)

1. Instalar Node.js y PM2:

    - Descarga e instala `nvm`:

        ```bash
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
        ```

    - En lugar de reiniciar la shell, ejecuta:

        ```bash
        \. "$HOME/.nvm/nvm.sh"
        ```

    - Descarga e instala `Node.js`:

        ```bash
        nvm install 22
        ```
    
    - Verifica la versión de `Node.js`:

        ```bash
        node -v 
        ```
        > Debería mostrar por ejemplo: "v22.20.0".

    - Verifica versión de `npm`:

        ```bash
        npm -v
        ```
        > Debería mostrar por ejemplo: "10.9.3"

2. **Clonar aplicación del repositorio git:**

    ```bash
    git clone ...
    ```

3. **Lanzar Instancias con PM2:**

    - Lanzar App 1 en puerto 3001

        ```bash
        pm2 start app.js --name "app_v3001" -- env PORT=3001
        ```

    - Lanzar App 2 en puerto 3002

        ```bash
        pm2 start app.js --name "app_v3002" -- env PORT=3002
        ```

    - Configurar auto-arranque

        ```bash
        pm2 startup
        ```

### Ejercicio 3: Servidor de Base de Datos (VM 3)

1. **Instalar MariaDB:**

    ```bash
    sudo apt install mariadb-server -y
    ```

2. **Hardening:** 

    ```bash
    sudo mysql_secure_installation
    ```

3. **Configurar Acceso:**

    - Editar el archivo `50-server.cnf` para cambiar `bind-address` a la IP interna de la VM 3: `bind-address = 192.168.10.4`.

    - **Configurar UFW:** Permitir el puerto **3306** solo desde la VM 2 (`192.168.10.3`).

        ```bash
        sudo ufw allow proto tcp from 192.168.10.3 to any port 3306
        ```

4. **Crear BD y Usuario:** Crear la base de datos `app_db` y el usuario `app_user` con permisos restringidos.

### Ejercicio 4: Servicios de Monitoreo (VM 1)

1. **Instalar Prometheus y Grafana (en VM 1 - Proxy/Monitoring).**

2. **Instalar Node Exporter:** Instalar el agente en las **VMs 2 y 3** para exponer métricas del SO en el puerto `9100`.

3. **Configurar Prometheus:**

    - Editar `prometheus.yml` para incluir los targets de las VMs 2 y 3:

        ```yaml
        scrape_configs:
          - job_name: 'node_apps'
            static_configs:
              - targets: ['192.168.10.3:9100', '192.168.10.4:9100'] # Apps y DB
        ```

4. **Integrar y Visualizar:** Acceder a Grafana (vía el puerto `8080` de su PC anfitriona). Añadir Prometheus como fuente de datos y visualizar las métricas del sistema.

## ⚙️ Práctica en Grupo (Réplica en Centro de Datos)

El objetivo es aplicar los ejercicios individuales en un entorno de 4 Máquinas Virtuales (VMs) reales, cada una con su rol, VLAN y subred asignada por grupo.

1. **Asignación Dinámica de Subredes y VLANs**

    Cada grupo (G) obtendrá una subred /29 y una VLAN única, donde *N* es el número del grupo.

    - **Subred Asignada:** `192.168. (100 + N) .0 / 29`

    - **VLAN ID:** `100 + N`

    | Grupo N | Subred | VLAN ID | Rango de IPs de Host |
    | - | - | - | - |
    | **G=1** | **192.168.101.0/29** | 101 | 192.168.101.1 - 192.168.101.6 |
    | **G=2** | 192.168.102.0/29 | 102 | 192.168.102.1 - 192.168.102.6 |
    | **G=50** | 192.168.150.0/29 | 150 | 192.168.150.1 - 192.168.150.6 |

2. **Asignación de Roles por IP (Ejemplo Grupo N)**

    El docente proveerá la IP del **Gateway (GW)** (`192.168.(100+N).1`).

    | Rol en el Sistema | IP Asignada | Servidor (VM) | VLAN |
    | - | - | - | - |
    | **Gateway (GW) / Router**	| `192.168.(100+N).1` | Router de la Carrera | N |
    | **PROXY + MONITOREO** | `192.168.(100+N).2` | VM 1 | N |
    | **APLICACIÓN 1** | `192.168.(100+N).3` | VM 2 | N | 
    | **APLICACIÓN 2** | `192.168.(100+N).4` | VM 3 | N | 
    | **BASE DE DATOS** | `192.168.(100+N).5` | VM 4 | N | 

3. **Tareas Críticas de Configuración Grupal**

    1. **Configuración de VLANs (Netplan):** En cada VM, configurar la interfaz de red para usar el VLAN ID asignado por el router central (si aplica).

        ```yaml
        # Ejemplo en Netplan para la VM 1 (Proxy)
        network:
          version: 2
          renderer: networkd
          ethernets:
            enp0s3: {}
          vlans:
            vlan101: # Reemplazar 101 por VLAN ID (100+N)
              id: 101
              link: enp0s3
              addresses: [192.168.101.2/29]
              routes:
                - to: default
                  via: 192.168.101.1
        ```

    2. **Hardening Avanzado (UFW):**

        - **Proxy (VM 1):** Permitir tráfico saliente (`route allow`) a las IPs de las VMs 2, 3 y 4.

        - **DB (VM 4):** Permitir puerto 3306 solo desde las IPs de las VMs 2 y 3.

    3. **Ajuste del Proxy:** Modificar la configuración de Nginx en la VM 1 (Proxy) para que use las IPs reales de los servidores de aplicación:

        ```nginx
        upstream backend_apps {
            server 192.168.(100+N).3:3000;
            server 192.168.(100+N).4:3000;
        }
        ```
    4. **Monitoreo:** El servidor de Monitoreo debe ser capaz de acceder a los puertos 9100 y 3306 (si aplica) de todos los demás servidores para extraer métricas.

    5. **Demostración Final:** Se probará la caída de una aplicación (VM 3) y se verificará el failover automático, y la alerta en Grafana.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos, que demuestran el dominio de los conceptos y la correcta ejecución de los pasos.