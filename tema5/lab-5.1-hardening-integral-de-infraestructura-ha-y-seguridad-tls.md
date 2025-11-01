# Laboratorio 5.1: Hardening Integral de Infraestructura HA y Seguridad TLS

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

- Aplicar un proceso de **Hardening Integral** (Firewall, SSH, Kernel) a todos los nodos de la infraestructura de 3 capas (Proxy, Aplicaciones, Base de Datos).

- Implementar configuraciones avanzadas de seguridad (Hardening y Tunning) en el servidor web **Nginx** y el servicio de **Base de Datos**.

- Asegurar el tráfico mediante **SSL/TLS**, utilizando certificados autofirmados (Individual) y de origen Cloudflare (Grupal).

- Implementar **seguridad perimetral** forzando la restricción de acceso al Proxy solo desde el Gateway de la VLAN.


## 🛠️ Preparación del Entorno Virtual (Práctica Individual)

Se utilizarán las tres Máquinas Virtuales (VMs) del Laboratorio 4.1, simulando una infraestructura de tres capas aisladas.

| VM | Rol en la Arquitectura | Servicios a Asegurar | Requisitos de Acceso |
| - | - | - | - |
| VM 1 | Proxy/Load Balancer | Nginx, UFW, SSH | Acceso desde Host (80, 443, 2222) |
| VM 2 | Servidor de Aplicaciones | Aplicación (ej., PHP-FPM), UFW, SSH | Acceso solo desde VM 1 (Proxy) y Host (SSH) |
| VM 3 | Servidor de Base de Datos | MariaDB Server, UFW, SSH | Acceso solo desde VM 2 (App) y Host (SSH) |

## 💻 Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Hardening Básico del Sistema Operativo y SSH (en las 3 VMs)

Esta configuración debe replicarse en las tres VMs (Proxy, App, DB). El objetivo es fortalecer la base del sistema operativo y asegurar el principal vector de acceso remoto (SSH), aplicando el principio de mínimo privilegio en cada servidor.

1. **Hardening de SSH (Acceso Seguro):**

    - Editar el archivo de configuración de SSH

        ```bash
        sudo nano /etc/ssh/sshd_config
        ```

    - Asegurarse de que las siguientes líneas estén configuradas

        ```nano
        PermitRootLogin no
        Port 2222 # puedes utilizar otro puerto si deseas
        ```

        > **Explicación:** Se modifican tres directivas clave: `PermitRootLogin no` elimina el usuario más poderoso como punto de entrada de ataques de fuerza bruta. `Port 2222` oculta el puerto SSH por defecto (ofuscación).
    
    - Aplicar cambios

        ```bash
        sudo systemctl restart ssh
        ```

        ```bash
        sudo reboot # En caso de que los cambios no se apliquen
        ```

2. **Configuración de Firewall (UFW):**

    - Instalar UFW por si no estuviese instalado

        ```bash
        sudo apt install ufw
        ```

    - Añadir política "Denegar por Defecto" (en todas las VMs)

        ```bash
        sudo ufw default deny incoming
        ```

        ```bash
        sudo ufw default allow outgoing
        ```

        > **Explicación:** Se aplica el principio de "**Denegar por Defecto**" (`deny incoming`). El sistema bloqueará todo el tráfico de entrada a menos que se cree explícitamente una regla de permiso.
    
    - Añadir permisos específicos (Ejemplo de VM 1: Proxy)

        ```bash
        sudo ufw allow 80/tcp
        ```

        ```bash
        sudo ufw allow 443/tcp
        ```

        ```bash
        sudo ufw allow 3000/tcp # Habilitar para Grafana
        ```

        ```bash
        sudo ufw allow 2222/tcp  # Nuevo puerto SSH
        ```

        > **Explicación:** Se habilitan los puertos específicos que serán utilizados para comunicarse con el Proxy

    - Añadir permisos específicos (Ejemplo de VM 2: App - Solo desde Proxy)

        ```bash
        sudo ufw allow from 192.168.10.2 to any port 2222 # Puerto de la App 1
        ```

        ```bash
        sudo ufw allow from 192.168.10.2 to any port 3001 # Puerto de la App 1
        ```

        ```bash
        sudo ufw allow from 192.168.10.2 to any port 3002 # Puerto de la App 2
        ```

        > **Explicación:** El acceso se restringe por IP de origen. Esto garantiza que si el Proxy (VM 1) es comprometido, el atacante no pueda acceder directamente a a la App (VM 2).

    - Añadir permisos específicos (Ejemplo de VM 3: DB - Solo desde App)

        ```bash
        sudo ufw allow from 192.168.10.2 to any port 2222
        ```

        ```bash
        sudo ufw allow from 192.168.10.3 to any port 3306
        ```

        > **Explicación:** El acceso se restringe por IP de origen. Esto garantiza que si el Proxy (VM 1) es comprometido, el atacante no pueda acceder directamente a la DB desde otra máquina, y que la DB solo hable con el App Server.

    - Habilitar UFW

        ```bash
        sudo ufw enable
        ```

        ```bash
        sudo ufw status verbose
        ```

### Ejercicio 2: Hardening y Tunning de Nginx (Solo en VM 1: Proxy/LB)

1. **Tunning (nginx.conf)**:

    ```bash
    sudo nano /etc/nginx/nginx.conf
    ```

2. **Modificar/añadir las siguientes configuraciones:**

    ```nano
    # En el bloque http { ...
    server_tokens off; 

    # En el bloque events { ...
    worker_connections 1024; 

    # Tunning de sesiones SSL
    ssl_session_cache shared:SSL:10m;
    ```

    > **Explicación:** La directiva `server_tokens off` elimina la información de la versión de Nginx y del sistema operativo en las respuestas HTTP. Esto es una forma de **ofuscación** que dificulta a los atacantes identificar vulnerabilidades específicas (*zero-day*) para esa versión. La directiva `worker_connections` define el número máximo de conexiones simultáneas (`1024`) que cada uno de sus procesos de trabajo puede manejar. Y `ssl_session_cache` crea una caché de sesiones SSL que se comparte entre los procesos de NGINX y puede almacenar hasta 10 megabytes de datos de sesión para acelerar las conexiones TLS/SSL.

3. **Rate Limiting (Control Anti-DDoS):**

    ```nano
    # 1. Definir la zona de límite en el bloque http { ...
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=5r/s;
    ```
    > **Explicación:** Se define un área de memoria (`zone=api_limit`) para rastrear peticiones. La regla es: **5 peticiones por segundo** (`5r/s`) por dirección IP única (`$binary_remote_addr`).

    ```nano
    # 2. Aplicar el límite en el bloque location / { ... del sitio
    limit_req zone=api_limit burst=10 nodelay;
    ```

    > **Explicación:** Aplica la zona de límite y permite un *burst* (ráfaga) de hasta 10 peticiones. Esto es crucial para mitigar ataques de Denegación de Servicio (DoS) al proteger la capa backend de ser saturada por peticiones maliciosas o automatizadas.

4. **Cabeceras de Seguridad:**

    ```nano
    # En el bloque server 443 { ...
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN";
    ```

    > **Explicación:** Se implementan **Cabeceras de Seguridad** para proteger a los usuarios. **HSTS** fuerza al navegador a usar siempre HTTPS, previniendo ataques de downgrade. `X-Frame-Options` previene el ataque de *Clickjacking*, que consiste en incrustar la página en un *iframe* malicioso.

### Ejercicio 3: Generación e Instalación de Certificado SSL/TLS Autofirmado (Solo en VM 1: Proxy/LB)

1. **Generar Clave y Certificado:**

    ```bash
    # Genera la clave privada (2048 bits) y el certificado autofirmado por 365 días
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/ssl/nginx-selfsigned.key \
    -out /etc/nginx/ssl/nginx-selfsigned.crt
    ```

    > **Explicación:** `openssl req -x509` se utiliza para crear un certificado autofirmado. Esto simula un entorno seguro de *backend* o de desarrollo donde la comunicación debe ser cifrada, pero no se requiere (ni se pagará) por la validación de una CA pública. *-nodes* asegura que la clave no esté protegida por contraseña, facilitando la automatización.

2. **Hardening TLS:**

    - Modificar el archivo `/etc/nginx/sites-available/default`.

    ```bash
    # En el bloque server 443 { ...
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
    ```

    > **Explicación:** Se fuerzan las versiones modernas de TLS (TLSv1.2 y TLSv1.3) y se especifica un conjunto de cifrados robustos (ssl_ciphers). Esto previene el uso de algoritmos obsoletos y vulnerables (como SSLv3 o TLSv1.0), que podrían ser explotados en ataques de criptografía.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual: 70 pts 

- Informe detallado con capturas de pantalla que demuestren ambas prácticas: 30 pts