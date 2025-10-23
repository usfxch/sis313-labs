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


## 💻 Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Proxy Inverso y Balanceo de Carga (VM Lab4.1-Proxy)

1. **Configurar IPs Estáticas:** Asignar las IPs `192.168.10.2` (interna) a la VM `Lab4.1-Proxy` y las IPs correspondientes a las VMs 2 y 3 en Netplan.

2. **Instalar Nginx:**

    ```bash
    sudo apt install nginx -y
    ```

3. **Configurar el Balanceador:**

    - Configurar el servidor y crear el bloque `upstream` en `/etc/nginx/sites-available/default` con las IPs de la VM `Lab4.1-Apps` (donde correrán las dos apps, en diferentes puertos).

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
        sudo apt install iptables-persistent
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

2. Instalar Node.js (instrucciones de [Web oficial de Node.js] (https://nodejs.org/es/download)):

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

3. Instalar PM2 (instrucciones de [Web oficial de PM2](https://pm2.keymetrics.io/docs/usage/quick-start/)):

    - Instala el administrador de procesos `PM2` de forma global en tu sistema.

        ```bash
        npm install pm2@latest -g
        ```
    - Verifica la versión de PM2 instalada:

        ```bash
        pm2 --version
        ```
        > Debería mostrar por ejemplo: "6.0.13"

4. **Clonar aplicación del repositorio git:**

    - Crea una carpeta para organizar las Apps:

        ```bash
        mkdir apps
        ```

    - Clona el proyecto para la App 1:

        ```bash
        git clone https://github.com/marceloquispeortega/api-restful-crud-movies app1_3001
        ```

    - Clona el proyecto para la App 2:
        ```bash
        git clone https://github.com/marceloquispeortega/api-restful-crud-movies app2_3002
        ```

    - Ya dentro de cada una de las carpetas, instala las dependencias:

        ```bash
        npm install
        ```

### Ejercicio 3: Servidor de Base de Datos (VM Lab4.2-DB)

1. En caso de haber clonado la VM, puedes cambiar el `hostname` utilizando el siguiente comando:

    ```bash
    sudo hostnamectl set-hostname db
    ```
    > Donde `db` es el nuevo `hostname` del VM.

2. **Instalar MariaDB:**

    ```bash
    sudo apt install mariadb-server -y
    ```

3. **Hardening:**: sigue los pasos para asegurar el servicio de MariaDB.

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

4. **Configurar Acceso:**

    - Editar el archivo `50-server.cnf` para cambiar `bind-address` a la IP interna de la VM Lab4.1-DB: `bind-address = 192.168.10.4`.

        ```bash
        sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
        ```

        > Dentro del archivo remplaza `bind-address = 127.0.0.1` por `bind-address = 192.168.10.4`.

    - Reinicia el servicio de MariaDB para aplicar cambios:

        ```bash
        sudo systemctl restart mariadb
        ```

5. **Crear BD y Usuario:** Crear la base de datos `db_movies` y el usuario `usr_movies` con permisos solo para esa base de datos.

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
        mysql -u root -h localhost -p
        ```

    - Selecciona la base de datos `db_movies`:

         ```bash
        USE db_movies
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

- Dentro de cada App debes copiar el archivo `.env.example` a `.env`. Asegurate que el archivo de App 1 tenga el puerto `3001` y el de App 2 tenga el puerto `3002`.

- Prueba si todo está correcto en ambas Apps utilizando el comando `node`:

    ```bash
    node app.js
    ```

    El mensaje que debería salir en consola debe ser el siguiente:

    <pre>
    Servidor ejecutándose en el puerto 3001
    Conexión a MariaDB exitosa. Pool creado y probado.</pre>

- Lanzar App 1 en puerto 3001

    ```bash
    pm2 start app.js --name app1_3001
    ```

- Lanzar App 2 en puerto 3002

    ```bash
    pm2 start app.js --name app2_3002
    ```

- Configurar auto-arranque

    ```bash
    pm2 startup
    ```

    ```bash
    pm2 save
    ```

### Ejercicio 5: Servicios de Monitoreo (VM Lab4.1-Proxy)

1. **Instalar Prometheus y Grafana (en VM Lab4.1-Proxy - Proxy/Monitoring).**

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

2. **Instalar Node Exporter:** Instalar el agente en las **VMs Lab4.1-Proxy, Lab4.1-Apps y Lab4.1-DB** para exponer métricas del SO en el puerto `9100`.

    ```bash
    sudo apt install prometheus prometheus-node-exporter
    ```

    > Comprueba desde cada VM que puedes acceder a las métricas: `curl http://localhost:9100/metrics`

4. **Integrar y Visualizar:** Acceder a Grafana (vía el puerto `8080` de su PC anfitriona). 

    - Accede con usuario `admin` y contraseña `admin`. Luego de ingresar, te pedirá poner una nueva contraseña.

    - Añade las fuentes de datos (`data source`) de `Prometheus` de cada VM. Utiliza los nombres `prometheus-proxy` (`http://192.168.10.2:9090`), `prometheus-apps` (`http://192.168.10.3:9090`) y `prometheus-db` (`http://192.168.10.4:9090`) según la VM que corresponda.

    - Importa los Dashboard `Node Exporter Full` (ID: `1860`) y `Node Exporter Full with Node Name` (ID: `10242`) para que puedas monitorear el estado de cada VM y visualizar las métricas del sistema.

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

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual: 35 pts 

- Desarrollo de la práctica grupal: 35 pts

- Informe detallado con capturas de pantalla que demuestren ambas prácticas: 30 pts