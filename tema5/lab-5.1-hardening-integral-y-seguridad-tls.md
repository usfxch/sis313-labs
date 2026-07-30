# Laboratorio 5.1: Hardening Integral y Seguridad TLS

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Aplicar hardening integral** a servidores Linux (SSH, Firewall, Kernel).

- **Implementar SSL/TLS** utilizando certificados autofirmados para asegurar el tráfico web.

- **Configurar segmentación de red** mediante UFW con política de denegar por defecto.

- **Implementar cabeceras de seguridad HTTP** (HSTS, X-Frame-Options) y hardening de Nginx.

- **Comprender la defensa en profundidad** aplicando múltiples capas de seguridad en una infraestructura de servicios.

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando **3 Máquinas Virtuales (VMs)** con **Ubuntu Server 24.04 LTS**, simulando una infraestructura segmentada con acceso restringido.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.20.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab5.1-Web` | `web` | **SERVIDOR WEB (Nginx + TLS)** | NAT (Internet) + Red Interna | `192.168.20.2` |
    | `Lab5.1-DB` | `db` | **SERVIDOR DE BASE DE DATOS (MariaDB)** | Red Interna | `192.168.20.3` |
    | `Lab5.1-Client` | `client` | **CLIENTE DE PRUEBAS** | Red Interna | `192.168.20.4` |

    - **Gateway (GW):** La interfaz interna de la VM `Lab5.1-Web` actuará como puerta de enlace para el tráfico de salida de las VMs `DB` y `Client`.

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab5_1`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab5.1-Web (Servidor Web):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab5_1`).

        - **VM Lab5.1-DB (Base de Datos):**

            - Adaptador 1: **Red Interna** (`Red_Lab5_1`).

        - **VM Lab5.1-Client (Cliente):**

            - Adaptador 1: **Red Interna** (`Red_Lab5_1`).

    3. **Reenvío de Puertos (Port Forwarding) en VM Lab5.1-Web (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab5.1-Web` para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP | 127.0.0.1 | **2222** | 10.0.2.15 | 22 | Acceso Remoto |
        | **HTTPS** | TCP | 127.0.0.1 | **4443** | 10.0.2.15 | 443 | Acceso Web Seguro |
        | **HTTP** | TCP | 127.0.0.1 | **8080** | 10.0.2.15 | 80 | Acceso Web (redirección) |

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Configuración de Red Estática (en las 3 VMs)

1. **Configurar IP estática en la VM Web:**

    Edita el archivo de configuración de red:

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
            - 192.168.20.2/29
          nameservers:
            addresses:
              - 8.8.8.8
    ```

    Aplica los cambios:

    ```bash
    sudo netplan apply
    ```

2. **Configurar IP estática en la VM DB:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.20.3/29
          nameservers:
            addresses:
              - 192.168.20.2
          routes:
            - to: default
              via: 192.168.20.2
    ```

    > **Explicación:** El servidor Web (`192.168.20.2`) también actúa como gateway. La VM DB no tiene acceso directo a Internet.

3. **Configurar IP estática en la VM Cliente:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.20.4/29
          nameservers:
            addresses:
              - 192.168.20.2
          routes:
            - to: default
              via: 192.168.20.2
    ```

4. **Habilitar reenvío de paquetes en la VM Web:**

    ```bash
    sudo nano /etc/sysctl.conf
    ```

    Asegúrate de que esta línea esté presente y descomentada:

    ```bash
    net.ipv4.ip_forward=1
    ```

    Aplica los cambios:

    ```bash
    sudo sysctl -p
    ```

    Configura `iptables` para NAT:

    ```bash
    sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

### Ejercicio 2: Hardening de SSH (en VM Web y VM DB)

El servicio SSH es el principal vector de ataque. Se aplicarán medidas de fortalecimiento en ambos servidores.

1. **Generar par de claves SSH (desde tu PC anfitriona o VM Cliente):**

    ```bash
    ssh-keygen -t ed25519 -C "usuario@lab51"
    ```

    Copia la clave pública a los servidores (mientras aún funcione el puerto 22):

    ```bash
    ssh-copy-id -p 2222 usuario@127.0.0.1  # VM Web vía port forwarding
    ```

2. **Editar configuración SSH en la VM Web:**

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    Modifica o asegúrate de que contengan las siguientes líneas:

    ```bash
    Port 2222
    PermitRootLogin no
    PasswordAuthentication no
    PubkeyAuthentication yes
    MaxAuthTries 3
    AllowUsers usuario
    ```

    > **Explicación:**
    > - `Port 2222`: Ofusca el servicio moviéndolo del puerto por defecto.
    > - `PermitRootLogin no`: Elimina el usuario root como vector de ataque.
    > - `PasswordAuthentication no`: Fuerza el uso de claves públicas, mucho más seguras.
    > - `MaxAuthTries 3`: Limita intentos de autenticación fallidos.
    > - `AllowUsers`: Solo usuarios explícitamente listados pueden conectarse.

3. **Reiniciar SSH y verificar:**

    ```bash
    sudo systemctl restart sshd
    sudo systemctl status sshd
    ```

    Prueba la conexión desde la VM Cliente (o anfitrión):

    ```bash
    ssh -p 2222 usuario@192.168.20.2
    ```

4. **Repetir el proceso de hardening SSH en la VM DB.**

    > **Importante:** Abre una sesión SSH separada y verifica que funciona antes de cerrar la sesión actual. Si te bloqueas, perderás acceso a la VM.

### Ejercicio 3: Configuración de Firewall UFW (Segmentación de Red)

Se aplicará el principio de **denegar por defecto** y se permitirá únicamente el tráfico necesario.

1. **En la VM Web (Proxy/Servidor Web):**

    Instalar UFW si no está instalado:

    ```bash
    sudo apt update
    sudo apt install ufw -y
    ```

    Aplicar políticas por defecto:

    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```

    Permitir servicios necesarios:

    ```bash
    sudo ufw allow 2222/tcp   # SSH hardened
    sudo ufw allow 80/tcp     # HTTP
    sudo ufw allow 443/tcp    # HTTPS
    ```

    Habilitar UFW:

    ```bash
    sudo ufw enable
    sudo ufw status verbose
    ```

2. **En la VM DB (Base de Datos):**

    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```

    Permitir acceso SSH solo desde la red interna (o desde la IP del host):

    ```bash
    sudo ufw allow from 192.168.20.0/29 to any port 2222/tcp
    ```

    Permitir MariaDB **solo desde la VM Web**:

    ```bash
    sudo ufw allow from 192.168.20.2 to any port 3306/tcp
    ```

    Habilitar UFW:

    ```bash
    sudo ufw enable
    sudo ufw status verbose
    ```

    > **Explicación:** La VM DB solo acepta conexiones SSH desde la red interna y conexiones MySQL únicamente desde la VM Web (`192.168.20.2`). Ni el cliente ni Internet pueden acceder directamente a la base de datos.

3. **Pruebas de conectividad desde la VM Cliente:**

    Intenta hacer ping a ambos servidores:

    ```bash
    ping 192.168.20.2
    ping 192.168.20.3
    ```

    Intenta conectarte por SSH al puerto 22 (debe fallar):

    ```bash
    ssh usuario@192.168.20.2
    ```

    Intenta conectarte al puerto 2222 (debe funcionar desde la red interna):

    ```bash
    ssh -p 2222 usuario@192.168.20.2
    ```

    Intenta conectarte al puerto 3306 de la VM DB (debe fallar desde el cliente):

    ```bash
    nc -vz 192.168.20.3 3306
    ```

### Ejercicio 4: Kernel Hardening Básico (en VM Web y VM DB)

1. **Editar parámetros del kernel:**

    ```bash
    sudo nano /etc/sysctl.conf
    ```

    Agrega o asegúrate de que contengan:

    ```bash
    # Deshabilitar forwarding si no es router (solo en DB)
    net.ipv4.ip_forward=0

    # Prevención de IP spoofing
    net.ipv4.conf.all.rp_filter=1

    # Deshabilitar tecla SysRq
    kernel.sysrq=0

    # No generar dumps de procesos privilegiados
    fs.suid_dumpable=0
    ```

    Aplica los cambios:

    ```bash
    sudo sysctl -p
    ```

### Ejercicio 5: Instalación de Nginx y MariaDB

1. **En la VM Web, instalar Nginx:**

    ```bash
    sudo apt update
    sudo apt install nginx -y
    sudo systemctl enable --now nginx
    ```

    Crear un sitio de prueba:

    ```bash
    sudo mkdir -p /var/www/lab51.local
    sudo bash -c 'echo "<h1>Servidor Web Seguro - Lab 5.1</h1>" > /var/www/lab51.local/index.html'
    sudo bash -c 'echo "<p>Servido desde 192.168.20.2</p>" >> /var/www/lab51.local/index.html'
    ```

    Configurar virtual host:

    ```bash
    sudo nano /etc/nginx/sites-available/lab51.local
    ```

    ```nginx
    server {
        listen 80;
        server_name lab51.local www.lab51.local;
        root /var/www/lab51.local;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

    Activar el sitio:

    ```bash
    sudo ln -s /etc/nginx/sites-available/lab51.local /etc/nginx/sites-enabled/
    sudo rm /etc/nginx/sites-enabled/default
    sudo nginx -t
    sudo systemctl restart nginx
    ```

2. **En la VM DB, instalar MariaDB:**

    ```bash
    sudo apt update
    sudo apt install mariadb-server -y
    sudo systemctl enable --now mariadb
    ```

    Ejecutar el script de seguridad inicial:

    ```bash
    sudo mysql_secure_installation
    ```

    > Responde `Y` a todas las preguntas: cambiar contraseña root, eliminar usuario anónimo, deshabilitar root remoto, eliminar base de datos de prueba.

    Configurar `bind-address` para escuchar solo en la IP interna:

    ```bash
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
    ```

    Busca la línea `bind-address` y modifícala:

    ```ini
    bind-address = 192.168.20.3
    ```

    Reiniciar MariaDB:

    ```bash
    sudo systemctl restart mariadb
    ```

    > **Explicación:** `bind-address` restringe el servicio MariaDB para que solo escuche en la interfaz interna, nunca en `0.0.0.0`.

### Ejercicio 6: Generación e Instalación de Certificado SSL/TLS Autofirmado

1. **En la VM Web, crear directorio para certificados:**

    ```bash
    sudo mkdir -p /etc/nginx/ssl
    ```

2. **Generar clave privada y certificado autofirmado:**

    ```bash
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/nginx/ssl/nginx-selfsigned.key \
      -out /etc/nginx/ssl/nginx-selfsigned.crt \
      -subj "/C=BO/ST=Chuquisaca/L=Sucre/O=USFX/OU=SIS313/CN=lab51.local"
    ```

    > **Explicación:**
    > - `-x509`: Genera un certificado autofirmado (no una CSR).
    > - `-nodes`: La clave privada no tiene frase de paso (útil para automatización).
    > - `-days 365`: Vigencia de un año.
    > - `-subj`: Define los datos del certificado (país, organización, CN).

3. **Verificar el certificado generado:**

    ```bash
    sudo openssl x509 -in /etc/nginx/ssl/nginx-selfsigned.crt -text -noout | grep -E "Subject:|Issuer:|Not Before|Not After"
    ```

### Ejercicio 7: Configuración de Nginx con HTTPS y Hardening TLS

1. **Editar el virtual host para añadir HTTPS:**

    ```bash
    sudo nano /etc/nginx/sites-available/lab51.local
    ```

    Reemplaza el contenido por:

    ```nginx
    # Redirección HTTP a HTTPS
    server {
        listen 80;
        server_name lab51.local www.lab51.local;
        return 301 https://$server_name$request_uri;
    }

    # Servidor HTTPS
    server {
        listen 443 ssl;
        server_name lab51.local www.lab51.local;

        root /var/www/lab51.local;
        index index.html;

        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;

        # Hardening TLS
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 1d;

        # Cabeceras de seguridad
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;

        # Ocultar versión de Nginx
        server_tokens off;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

    > **Explicación de directivas:**
    > - `ssl_protocols TLSv1.2 TLSv1.3`: Fuerza versiones modernas y seguras de TLS.
    > - `ssl_ciphers`: Define cifrados robustos, excluyendo algoritmos débiles.
    > - `ssl_session_cache`: Mejora rendimiento reutilizando sesiones TLS.
    > - `HSTS`: Fuerza al navegador a usar siempre HTTPS.
    > - `X-Frame-Options`: Previene ataques de clickjacking.
    > - `server_tokens off`: Oculta la versión de Nginx (ofuscación).

2. **Verificar sintaxis y reiniciar Nginx:**

    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

3. **Verificar que Nginx escucha en 443:**

    ```bash
    sudo ss -tulnp | grep nginx
    ```

### Ejercicio 8: Pruebas de Acceso y Verificación de Seguridad

1. **Desde la VM Cliente, probar redirección HTTP → HTTPS:**

    ```bash
    curl -I http://192.168.20.2
    ```

    > **Resultado esperado:** Código `301 Moved Permanently` redirigiendo a HTTPS.

2. **Probar acceso HTTPS (ignorando advertencia de certificado autofirmado):**

    ```bash
    curl -k https://192.168.20.2
    ```

    > **Resultado esperado:** Deberías ver el HTML de bienvenida.

3. **Verificar las cabeceras de seguridad:**

    ```bash
    curl -k -I https://192.168.20.2
    ```

    > **Resultado esperado:** Deberías ver `strict-transport-security`, `x-frame-options`, `x-content-type-options`.

4. **Verificar la versión de TLS negociada:**

    ```bash
    openssl s_client -connect 192.168.20.2:443 -tls1_2 </dev/null
    ```

    ```bash
    openssl s_client -connect 192.168.20.2:443 -tls1_1 </dev/null
    ```

    > **Resultado esperado:** TLS 1.2 funciona. TLS 1.1 debería fallar (handshake error).

5. **Desde la PC Anfitriona (usando Port Forwarding):**

    Accede desde tu navegador a:

    ```
    https://127.0.0.1:4443
    ```

    > **Nota:** El navegador mostrará una advertencia de "Conexión no segura" porque el certificado es autofirmado. Esto es normal y esperado. Añade una excepción o haz clic en "Avanzado → Continuar".

6. **Verificar que la VM DB no es accesible desde el Cliente en el puerto 3306:**

    ```bash
    nc -vz -w 2 192.168.20.3 3306
    ```

    > **Resultado esperado:** `Connection timed out` (UFW está bloqueando).

## ⚙️ Sección 3: Práctica en Grupo (Red entre Pares)

El objetivo es que cada grupo de **2 estudiantes** despliegue una infraestructura con **hardening completo** y **comunicación cifrada TLS**, de modo que ambas PCs anfitrionas puedan verificar la seguridad mutuamente.

### 1. Roles y Asignación por Grupo

Cada grupo está compuesto por 2 integrantes.

| Integrante | Rol Principal | VM a Configurar | Requisitos de Red |
| - | - | - | - |
| **Integrante 1** | **Servidor Web Seguro** | VM con Nginx + TLS + SSH Hardened | Adaptador en modo **Puente (Bridge)** |
| **Integrante 2** | **Servidor de Base de Datos** | VM con MariaDB + SSH Hardened + UFW | Adaptador en modo **Puente (Bridge)** |

> **Nota:** El modo **Puente (Bridge)** permite que las VMs obtengan IPs directamente de la red física del laboratorio.

### 2. Dominio Local del Grupo

Cada grupo elegirá un dominio local único. Ejemplos:

- `seguro-grupo1.local`
- `lab5.corp`
- `hard.sis313`

> **Restricción:** El dominio debe ser único dentro del aula para evitar colisiones.

### 3. Tareas del Integrante 1 (Servidor Web)

1. **Configurar la VM en modo Puente** y asignar una IP estática dentro del rango de la red del laboratorio.

2. **Aplicar hardening SSH:** Puerto 2222, deshabilitar root, autenticación por clave pública.

3. **Instalar y configurar Nginx** con un sitio HTML estático que incluya:
    - Nombre del grupo.
    - Nombres de los integrantes.
    - Un mensaje indicando que el sitio está protegido con TLS.

4. **Generar certificado autofirmado** para el dominio del grupo.

5. **Configurar HTTPS** forzando TLS 1.2/1.3 y añadiendo cabeceras de seguridad (HSTS, X-Frame-Options).

6. **Configurar UFW:** Permitir 80, 443, 2222; denegar todo lo demás.

### 4. Tareas del Integrante 2 (Base de Datos)

1. **Configurar la VM en modo Puente** y asignar una IP estática.

2. **Aplicar hardening SSH** igual que el Integrante 1.

3. **Instalar MariaDB** y ejecutar `mysql_secure_installation`.

4. **Configurar `bind-address`** para escuchar solo en la IP interna.

5. **Configurar UFW:**
    - Permitir SSH (puerto 2222) solo desde la red del aula o desde la IP del Integrante 1.
    - Permitir MariaDB (puerto 3306) **solo desde la IP del Servidor Web** del Integrante 1.
    - Denegar todo lo demás.

6. **Crear un usuario de base de datos** específico para la aplicación (no usar root).

### 5. Pruebas de Seguridad Cruzadas

Cada integrante debe probar la infraestructura del otro grupo o de su compañero.

1. **Verificación de TLS (desde PC anfitriona o VM Cliente):**

    ```bash
    openssl s_client -connect IP_DEL_COMPAÑERO:443 -tls1_2
    ```

    Verificar que TLS 1.0/1.1 estén rechazados.

2. **Verificación de cabeceras de seguridad:**

    ```bash
    curl -k -I https://IP_DEL_COMPAÑERO
    ```

    Debe mostrar `strict-transport-security` y `x-frame-options`.

3. **Verificación de segmentación (desde tu PC):**

    Intentar conectarte al puerto 3306 de la VM DB del compañero:

    ```bash
    nc -vz IP_DB_COMPAÑERO 3306
    ```

    > **Resultado esperado:** Debe fallar (timeout o refused) porque UFW solo permite conexiones desde el servidor web de su grupo.

4. **Verificación de ofuscación SSH:**

    Intentar conectarte al puerto 22 (debe fallar o no responder):

    ```bash
    ssh usuario@IP_COMPAÑERO
    ```

    Conectarte al puerto 2222 con clave pública (debe funcionar):

    ```bash
    ssh -p 2222 usuario@IP_COMPAÑERO
    ```

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual (hardening SSH, UFW, TLS, Nginx): 35 pts
- Desarrollo de la práctica grupal (segmentación, pruebas cruzadas): 35 pts
- Informe detallado con capturas de pantalla que demuestren ambas prácticas: 30 pts
