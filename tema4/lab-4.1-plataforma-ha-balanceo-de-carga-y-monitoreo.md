# Laboratorio 4.1: Plataforma HA, Balanceo de Carga y Monitoreo

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Implementar una arquitectura web de Alta Disponibilidad (HA)** con balanceo de carga, segregación de servicios y monitoreo integral de rendimiento.

- **Configurar un Proxy Inverso (Nginx)** para balancear el tráfico entre dos servidores de aplicaciones.

- **Desplegar dos instancias de aplicación (Node.js)** gestionadas por PM2 en modo de producción.

- **Instalar y configurar una Base de Datos centralizada (MariaDB)** y asegurar su acceso.

- **Integrar un sistema de Monitoreo (Prometheus + Grafana)** para medir la disponibilidad y el rendimiento de la plataforma.

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando 3 Máquinas Virtuales (VMs) con **Ubuntu Server 24.04 LTS**, simulando la segregación con interfaces de red y reenvío de puertos.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.10.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab4.1-Proxy` | `proxy` | **PROXY + MONITORING** | NAT (Internet) + Red Interna | `192.168.10.2`|
    | `Lab4.1-Apps` | `apps` | **APLICACIONES (Apps 1 y 2)** | Red Interna | `192.168.10.3` |
    | `Lab4.1-DB` | `db` | **BASE DE DATOS (DB)** | Red Interna | `192.168.10.4` |

    - **Gateway (GW)**: La interfaz interna de la VM `Lab4.1-Proxy` actuará como router/puerta de enlace para el tráfico de salida de las VMs 2 y 3. (Se configura a mano el *binding*).

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab4_1`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab4.1-Proxy (Proxy/Monitoreo):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab4_1`).

        - **VM Lab4.1-Apps (Aplicaciones):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_1`).

        - **VM Lab4.1-DB (Base de Datos):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_1`).
    
    3. **Reenvío de Puertos (Port Forwarding) en VM Lab4.1-Proxy (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab4.1-Proxy` **(Proxy/Monitoring)** para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP	| 127.0.0.1	| **2222** | 10.0.2.15 | 22 | Acceso Remoto |
        | **PROXY** | TCP | 127.0.0.1 | **80** | 10.0.2.15 | 80 | Acceso Web (Nginx) |
        | **GRAFANA** | TCP | 127.0.0.1 | **8080** | 10.0.2.15 | 3000 | Monitoreo Web |

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Proxy Inverso y Balanceo de Carga (VM Lab4.1-Proxy)

1. **Configurar IPs Estáticas:**

    - Asigna las IPs estáticas a la VM `Lab4.1-Proxy` y las IPs correspondientes a las VMs `Lab4.1-Apps` y `Lab4.1-DB` en Netplan.

    - Edita el archivo de configuración de red en la VM `Lab4.1-Proxy`:

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
                - 192.168.10.2/29
              nameservers:
                addresses:
                  - 8.8.8.8
        ```

    - Aplica la configuración de red:

        ```bash
        sudo netplan apply
        ```

2. **Instalar Nginx:**

    ```bash
    sudo apt update && sudo apt install nginx -y
    ```

3. **Configurar el Balanceador:**

    - Configura el servidor y crea el bloque `upstream` en `/etc/nginx/sites-available/default` con las IPs de la VM `Lab4.1-Apps` (donde correrán las dos apps, en diferentes puertos).

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
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
        ```

    - Verifica que la configuración de Nginx sea válida:

        ```bash
        sudo nginx -t
        ```

    - Reinicia el servicio de Nginx para aplicar los cambios:

        ```bash
        sudo systemctl restart nginx
        ```

4. **Enrutamiento y NAT:**

    - Habilitar el reenvío de paquetes en el kernel

        Edita el archivo `sysctl.conf`:
        ```bash
        sudo nano /etc/sysctl.conf
        ```
    
        Asegúrate de que esta línea esté presente y no comentada:
        ```bash
        net.ipv4.ip_forward=1
        ```

        Aplicar los cambios inmediatamente
        ```bash
        sudo sysctl -p
        ```

    - Configura las reglas de `iptables` para redirigir el tráfico de la red interna (`enp0s8`) a través de la interfaz que tiene acceso a internet (`enp0s3`).

        ```bash
        sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
        ```

    - Guarda las reglas para que persistan después de reiniciar el servidor.

        ```bash
        sudo apt install iptables-persistent -y
        ```

        ```bash
        sudo netfilter-persistent save
        ```

### Ejercicio 2: Servidores de Aplicaciones (VM Lab4.1-Apps)

1. En caso de haber clonado la VM, puedes cambiar el `hostname` utilizando el siguiente comando:

    ```bash
    sudo hostnamectl set-hostname apps
    ```
    > Donde `apps` es el nuevo `hostname` del VM.

2. **Configurar IP Estática:**

    - Edita el archivo de configuración de red en la VM `Lab4.1-Apps`:

        ```bash
        sudo nano /etc/netplan/50-cloud-init.yaml
        ```

        ```yaml
        network:
          version: 2
          ethernets:
            enp0s3:
              dhcp4: no
              optional: true
              addresses:
                - 192.168.10.3/29
              routes:
                - to: default
                  via: 192.168.10.2
              nameservers:
                addresses:
                  - 8.8.8.8
        ```

    - Aplica la configuración de red:

        ```bash
        sudo netplan apply
        ```

3. Instalar Node.js (instrucciones de [Web oficial de Node.js] (https://nodejs.org/es/download)):

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

4. Instalar PM2 (instrucciones de [Web oficial de PM2](https://pm2.keymetrics.io/docs/usage/quick-start/)):

    - Instala el administrador de procesos `PM2` de forma global en tu sistema.

        ```bash
        npm install pm2@latest -g
        ```
    - Verifica la versión de PM2 instalada:

        ```bash
        pm2 --version
        ```
        > Debería mostrar por ejemplo: "6.0.13"

5. **Clonar aplicación del repositorio git:**

    - Crea una carpeta para organizar las Apps:

        ```bash
        mkdir ~/apps && cd ~/apps
        ```

    - Clona el proyecto para la App 1:

        ```bash
        git clone https://github.com/marceloquispeortega/api-restful-crud-movies app1_3001
        ```

    - Clona el proyecto para la App 2:

        ```bash
        git clone https://github.com/marceloquispeortega/api-restful-crud-movies app2_3002
        ```

    - Dentro de cada una de las carpetas, instala las dependencias:

        ```bash
        cd ~/apps/app1_3001 && npm install
        cd ~/apps/app2_3002 && npm install
        ```

6. **Configurar variables de entorno:**

    - Dentro de cada App, copia el archivo de ejemplo a `.env`:

        ```bash
        cd ~/apps/app1_3001 && cp .env.example .env
        cd ~/apps/app2_3002 && cp .env.example .env
        ```

    - Edita el archivo `.env` de la **App 1** (`~/apps/app1_3001/.env`) y asegúrate de que contenga los siguientes valores:

        ```bash
        PORT=3001
        DB_HOST=192.168.10.4
        DB_USER=usr_movies
        DB_PASSWORD=secret
        DB_NAME=db_movies
        ```

    - Edita el archivo `.env` de la **App 2** (`~/apps/app2_3002/.env`) y asegúrate de que contenga los siguientes valores:

        ```bash
        PORT=3002
        DB_HOST=192.168.10.4
        DB_USER=usr_movies
        DB_PASSWORD=secret
        DB_NAME=db_movies
        ```

### Ejercicio 3: Servidor de Base de Datos (VM Lab4.1-DB)

1. En caso de haber clonado la VM, puedes cambiar el `hostname` utilizando el siguiente comando:

    ```bash
    sudo hostnamectl set-hostname db
    ```
    > Donde `db` es el nuevo `hostname` del VM.

2. **Configurar IP Estática:**

    - Edita el archivo de configuración de red en la VM `Lab4.1-DB`:

        ```bash
        sudo nano /etc/netplan/50-cloud-init.yaml
        ```

        ```yaml
        network:
          version: 2
          ethernets:
            enp0s3:
              dhcp4: no
              optional: true
              addresses:
                - 192.168.10.4/29
              routes:
                - to: default
                  via: 192.168.10.2
              nameservers:
                addresses:
                  - 8.8.8.8
        ```

    - Aplica la configuración de red:

        ```bash
        sudo netplan apply
        ```

3. **Instalar MariaDB:**

    ```bash
    sudo apt install mariadb-server -y
    ```

4. **Hardening:** sigue los pasos para asegurar el servicio de MariaDB.

    ```bash
    sudo mysql_secure_installation
    ```

    <pre>
    Enter current password for root (enter for none): ⮐</pre>

    <pre>
    Switch to unix_socket authentication [Y/n] ⮐</pre>

    <pre>
    Change the root password? [Y/n] ⮐</pre>

    <pre>
    New password: (introduce tu contraseña) ⮐</pre>

    <pre>
    Re-enter new password:: (vuelve a introducir tu contraseña) ⮐</pre>

    <pre>
    Remove anonymous users? [Y/n] ⮐</pre>

    <pre>
    Disallow root login remotely? [Y/n] ⮐</pre>

    <pre>
    Remove test database and access to it? [Y/n] ⮐</pre>

    <pre>
    Reload privilege tables now? [Y/n] ⮐</pre>

    Intenta conectarte al servidor de MariaDB para probar que todo esté correcto:

    ```bash
    mysql -u root -h localhost -p
    ```

    <pre>
    Enter password: (introduce tu contraseña) ⮐</pre>

    <pre>
    MariaDB [(none)]> quit ⮐</pre>

5. **Configurar Acceso:**

    - Editar el archivo `50-server.cnf` para cambiar `bind-address` a la IP interna de la VM Lab4.1-DB: `bind-address = 192.168.10.4`.

        ```bash
        sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
        ```

        > Dentro del archivo remplaza `bind-address = 127.0.0.1` por `bind-address = 192.168.10.4`.

    - Reinicia el servicio de MariaDB para aplicar cambios:

        ```bash
        sudo systemctl restart mariadb
        ```

6. **Crear BD y Usuario:** Crear la base de datos `db_movies` y el usuario `usr_movies` con permisos solo para esa base de datos.

    - Accede al CLI de MariaDB:

        ```bash
        mysql -u root -h localhost -p
        ```

    - Crea la base de datos:

        ```mysql
        CREATE DATABASE db_movies;
        ```

    - Crea el usuario:

        ```mysql
        CREATE USER 'usr_movies'@'192.168.10.3' IDENTIFIED BY 'secret';
        ```

    - Asigna todos los permisos al usuario para la base de datos:

        ```mysql
        GRANT ALL PRIVILEGES ON db_movies.* TO 'usr_movies'@'192.168.10.3';
        ```

    - Aplica cambios en MariaDB de los permisos asignados y sal del CLI de MariaDB:

        ```mysql
        FLUSH PRIVILEGES;
        ```

        ```mysql
        quit
        ```

    - Ingresa con el nuevo usuario:

        ```bash
        mysql -u usr_movies -h 192.168.10.4 -p
        ```

    - Selecciona la base de datos `db_movies`:

         ```mysql
        USE db_movies;
        ```
    
    - Ejecuta (copia y pega) los scripts de creación de la tabla movies y sus registros.

        ```sql
        CREATE TABLE movies (
            id serial PRIMARY KEY,
            title character varying(150) NOT NULL,
            year integer,
            UNIQUE(title)
        );
        
        INSERT INTO movies (title, year) VALUES
            ('Inception', 2010),
            ('The Matrix', 1999),
            ('Pulp Fiction', 1994),
            ('The Dark Knight', 2008),
            ('Eternal Sunshine of the Spotless Mind', 2004),
            ('Forrest Gump', 1994),
            ('Fight Club', 1999),
            ('The Godfather', 1972),
            ('Interstellar', 2014),
            ('Parasite', 2019);
        ```

    - Sal del CLI de MariaDB:

        ```mysql
        quit
        ```

### Ejercicio 4: Lanzar Instancias con PM2 (VM Lab4.1-Apps)

1. **Prueba manual de las aplicaciones:**

    - Prueba si todo está correcto en ambas Apps utilizando el comando `node`:

        ```bash
        cd ~/apps/app1_3001 && node app.js
        ```

        El mensaje que debería salir en consola debe ser el siguiente:

        <pre>
        Servidor ejecutándose en el puerto 3001
        Conexión a MariaDB exitosa. Pool creado y probado.</pre>

    - Detén la app con `Ctrl+C` y repite la prueba para la App 2:

        ```bash
        cd ~/apps/app2_3002 && node app.js
        ```

2. **Lanzar aplicaciones con PM2:**

    - Lanzar App 1 en puerto 3001

        ```bash
        cd ~/apps/app1_3001 && pm2 start app.js --name app1_3001
        ```

    - Lanzar App 2 en puerto 3002

        ```bash
        cd ~/apps/app2_3002 && pm2 start app.js --name app2_3002
        ```

    - Verifica el estado de los procesos:

        ```bash
        pm2 status
        ```

3. **Configurar auto-arranque:**

    ```bash
    pm2 startup
    ```

    ```bash
    pm2 save
    ```

4. **Verificar el Balanceo de Carga:**

    - Desde la VM `Lab4.1-Proxy`, realiza peticiones al balanceador y verifica que alterna entre las dos aplicaciones:

        ```bash
        curl http://192.168.10.2
        ```

    - Desde tu PC anfitriona (usando el reenvío de puertos configurado), accede al proxy en `http://127.0.0.1:80` o realiza pruebas con `curl`:

        ```bash
        curl http://127.0.0.1:80/movies
        ```

    - Realiza varias peticiones y verifica que el balanceador distribuye las solicitudes entre la App 1 y la App 2.

### Ejercicio 5: Servicios de Monitoreo (VM Lab4.1-Proxy)

1. **Instalar Grafana (en VM Lab4.1-Proxy - Proxy/Monitoring).**

    - Instale los paquetes de requisitos previos:

        ```bash
        sudo apt-get install -y apt-transport-https software-properties-common wget
        ```

    - Importar la clave GPG:

        ```bash
        sudo mkdir -p /etc/apt/keyrings/
        ```
        
        ```bash
        wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
        ```

    - Para agregar un repositorio para versiones estables, ejecute el siguiente comando:

        ```bash
        echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
        ```

    - Ejecute el siguiente comando para actualizar la lista de paquetes disponibles:

        ```bash
        sudo apt update
        ```

    - Para instalar Grafana OSS, ejecute el siguiente comando:

        ```bash
        sudo apt install grafana
        ```

        ```bash
        sudo systemctl daemon-reload
        ```

    - Habilitamos el servicio de Grafana:

        ```bash
        sudo systemctl enable grafana-server
        ```

        ```bash
        sudo systemctl start grafana-server
        ```

2. **Instalar Prometheus y Node Exporter:**

    - Instala Prometheus y el agente Node Exporter en la VM `Lab4.1-Proxy`:

        ```bash
        sudo apt install prometheus prometheus-node-exporter -y
        ```

    - Comprueba que puedes acceder a las métricas locales:

        ```bash
        curl http://localhost:9100/metrics
        ```

3. **Instalar Node Exporter en las VMs Lab4.1-Apps y Lab4.1-DB:**

    - En cada una de estas VMs, instala el agente para exponer métricas del SO en el puerto `9100`:

        ```bash
        sudo apt install prometheus-node-exporter -y
        ```

    - Comprueba desde cada VM que puedes acceder a las métricas:

        ```bash
        curl http://localhost:9100/metrics
        ```

4. **Configurar Prometheus para scrapear métricas:**

    - Edita el archivo de configuración de Prometheus en la VM `Lab4.1-Proxy`:

        ```bash
        sudo nano /etc/prometheus/prometheus.yml
        ```

    - Añade o modifica la sección `scrape_configs` para incluir los targets de todas las VMs:

        ```yaml
        scrape_configs:
          - job_name: 'prometheus'
            static_configs:
              - targets: ['localhost:9090']

          - job_name: 'node-proxy'
            static_configs:
              - targets: ['192.168.10.2:9100']

          - job_name: 'node-apps'
            static_configs:
              - targets: ['192.168.10.3:9100']

          - job_name: 'node-db'
            static_configs:
              - targets: ['192.168.10.4:9100']
        ```

    - Reinicia el servicio de Prometheus para aplicar los cambios:

        ```bash
        sudo systemctl restart prometheus
        ```

    - Verifica que Prometheus esté funcionando correctamente:

        ```bash
        sudo systemctl status prometheus
        ```

5. **Integrar y Visualizar en Grafana:**

    - Accede a Grafana desde tu PC anfitriona vía el puerto `8080` (`http://127.0.0.1:8080`).

    - Accede con usuario `admin` y contraseña `admin`. Luego de ingresar, te pedirá poner una nueva contraseña.

    - Añade la fuente de datos (`data source`) de `Prometheus`. Utiliza la URL `http://localhost:9090` (Prometheus está en la misma VM que Grafana).

    - Importa los Dashboard `Node Exporter Full` (ID: `1860`) y `Node Exporter Full with Node Name` (ID: `10242`) para que puedas monitorear el estado de cada VM y visualizar las métricas del sistema.

## ⚙️ Sección 3: Práctica en Grupo (Réplica en Centro de Datos)

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

    | Rol en el Sistema | IP Asignada | Servidor (VM) | VLAN ID |
    | - | - | - | - |
    | **Gateway (GW) / Router**	| `192.168.(100+N).1` | Server Docente | 100+N |
    | **PROXY + MONITOREO** | `192.168.(100+N).2` | VM Proxy (alumno 1) | 100+N |
    | **APLICACIÓN 1** | `192.168.(100+N).3` | VM App 1 (alumno 2) | 100+N | 
    | **APLICACIÓN 2** | `192.168.(100+N).4` | VM App 2 (alumno 3) | 100+N | 
    | **BASE DE DATOS** | `192.168.(100+N).5` | VM DB (alumno 4) | 100+N | 

3. **Tareas Críticas de Configuración Grupal**

    1. **Configuración de VLANs (Netplan):** En cada VM, configurar la interfaz de red para usar el VLAN ID asignado por el router central (si aplica).

        ```yaml
        # Ejemplo en Netplan para la VM Proxy (Proxy)
        network:
          version: 2
          renderer: networkd
          ethernets:
            ens18:
              dhcp4: no
              optional: true
              addresses: # Esto si gustan lo pueden comentar una vez que la VLAN esté funcionando
              - "192.168.100.202/24" # Esto si gustan lo pueden comentar una vez que la VLAN esté funcionando
          vlans:
            vlan101: # Reemplazar 101 por VLAN ID (100+N)
              id: 101
              link: ens18
              addresses:
                - "192.168.101.2/29"
              nameservers:
                addresses:
                - 8.8.8.8
              routes:
                - to: default
                  via: 192.168.101.1
        ```

    2. **Hardening (UFW):**

        - **DB (VM 4):** Permitir puerto 3306 solo desde las IPs de las VMs 2 y 3.

    3. **Ajuste del Proxy:** Modificar la configuración de Nginx en la VM Proxy para que use las IPs reales de los servidores de aplicación:

        ```nginx
        upstream loadbalancer {
            server 192.168.101.3:3000; # App 1, reemplazar 101 por VLAN ID (100+N)
            server 192.168.101.4:3000; # App 2, reemplazar 101 por VLAN ID (100+N)
        }

        server {
            listen 80;
            server_name _;

            location / {
                proxy_pass http://loadbalancer;
            }
        }
        ```
    4. **Monitoreo:** El servidor de Monitoreo debe ser capaz de acceder a los puertos 9100 de todos los demás servidores; es decir: a si mismo (Proxy), a las Apps (App 1 y App2) y a la BD, para extraer métricas.

    5. **Acceso a la Proxy y Grafana desde Internet:** Para acceder al Proxy y Grafana se han configurado rutas en el Servidor Docente. Para ello, deberás acceder a las siguientes rutas:
        
        - Proxy: https://vlan101-app.rootcode.com.bo (asociado a http://192.168.101.2:80). Reemplazar 101 por VLAN ID (100+N)

        - Grafana: https://vlan101-monitoring.rootcode.com.bo (asociado a http://192.168.101.2:3000). Reemplazar 101 por VLAN ID (100+N)

    6. **Demostración Final:** Se probará la caída de una aplicación (VM App 1 o VM App 2) y se verificará el failover automático del balanceador, y la alerta en Grafana.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se centrará en la correcta implementación de la arquitectura de alta disponibilidad, el funcionamiento del balanceo de carga y la integración del monitoreo. Se valorará la habilidad para resolver problemas y la comprensión de los conceptos clave.

1. **Configuración del Entorno y Red (10 pts)**

    - **Configuración de Red:** Demuestra que las tres VMs tienen conectividad entre sí y que la VM Proxy enruta correctamente el tráfico de salida de las VMs Apps y DB.

    - **Configuración de Nginx:** El proxy inverso y el balanceo de carga están correctamente configurados en la VM `Lab4.1-Proxy`.

2. **Despliegue de Aplicaciones y Base de Datos (20 pts)**

    - **Servidores de Aplicaciones:** Las dos instancias de la aplicación Node.js están corriendo correctamente bajo PM2 en la VM `Lab4.1-Apps`.

    - **Base de Datos:** La base de datos `db_movies` está creada y el usuario `usr_movies` tiene acceso restringido desde la IP de la VM de aplicaciones.

    - **Conectividad App-DB:** La aplicación se conecta exitosamente a MariaDB y el endpoint `/movies` responde correctamente.

3. **Monitoreo y Visualización (20 pts)**

    - **Prometheus:** El servidor de Prometheus está configurado para recolectar métricas de todas las VMs (Proxy, Apps y DB).

    - **Node Exporter:** Los agentes están instalados y exponiendo métricas en el puerto 9100 en cada VM.

    - **Grafana:** Los dashboards de Node Exporter están importados y muestran métricas de las tres VMs.

4. **Práctica en Grupo y Demostración (20 pts)**

    - **Configuración Grupal:** El grupo replica correctamente la arquitectura en el entorno del centro de datos con las VLANs y subredes asignadas.

    - **Failover y Balanceo:** Se demuestra que al detener una aplicación, el balanceador redirige el tráfico a la instancia disponible.

    - **Colaboración y Presentación:** La evaluación considerará la calidad del trabajo en equipo, la claridad de la presentación y la capacidad para explicar la configuración y las pruebas realizadas.

5. **Informe de Laboratorio (30 pts)**

    El informe debe realizarse en formato Markdown y debe ser detallado con capturas de pantalla que demuestren cada uno de los pasos realizados, incluyendo:

    - Capturas de la configuración de red en cada VM (comando `ip addr`).
    - Capturas de la configuración de Nginx y el resultado de `sudo nginx -t`.
    - Capturas del estado de las aplicaciones con `pm2 status`.
    - Capturas de la conexión a la base de datos y la creación de la tabla `movies`.
    - Capturas de las pruebas de balanceo de carga (respuestas alternadas entre App 1 y App 2).
    - Capturas de Grafana mostrando métricas de las tres VMs.
    - Conclusiones y reflexiones sobre la arquitectura implementada.
