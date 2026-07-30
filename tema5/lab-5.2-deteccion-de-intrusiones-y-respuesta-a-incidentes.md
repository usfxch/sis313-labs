# Laboratorio 5.2: Detección de Intrusiones y Respuesta a Incidentes

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Configurar Fail2ban** como sistema de prevención de intrusiones basado en host (HIPS) para proteger servicios SSH y web.

- **Simular ataques de fuerza bruta** y escaneos de puertos desde una VM atacante.

- **Analizar logs del sistema** (`auth.log`, `access.log`) para identificar patrones de ataque y actividad sospechosa.

- **Aplicar el ciclo de respuesta a incidentes:** identificar, contener, erradicar, recuperar y documentar.

- **Comprender la importancia de la auditoría** y la centralización de logs en la ciberseguridad.

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando **2 Máquinas Virtuales (VMs)** con **Ubuntu Server 24.04 LTS**, simulando un escenario de ataque/defensa.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.30.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab5.2-Server` | `server` | **SERVIDOR OBJETIVO** (Nginx, SSH, Fail2ban) | NAT (Internet) + Red Interna | `192.168.30.2` |
    | `Lab5.2-Attacker` | `attacker` | **MÁQUINA ATACANTE** (Herramientas de pentesting) | Red Interna | `192.168.30.3` |

    - **Gateway (GW):** La interfaz interna de la VM `Lab5.2-Server` actuará como puerta de enlace para la VM `Attacker`.

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab5_2`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab5.2-Server (Servidor Objetivo):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab5_2`).

        - **VM Lab5.2-Attacker (Atacante):**

            - Adaptador 1: **Red Interna** (`Red_Lab5_2`).

    3. **Reenvío de Puertos (Port Forwarding) en VM Lab5.2-Server (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab5.2-Server` para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP | 127.0.0.1 | **2222** | 10.0.2.15 | 22 | Acceso Remoto |
        | **HTTP** | TCP | 127.0.0.1 | **8080** | 10.0.2.15 | 80 | Acceso Web |

        > **Nota:** A diferencia del Laboratorio 5.1, en este escenario se deja el SSH en el puerto 22 internamente (sin hardening de puerto) para facilitar la simulación del ataque. El port forwarding usa 2222 solo para acceso del anfitrión.

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Configuración de Red Estática

1. **Configurar IP estática en la VM Server:**

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
            - 192.168.30.2/29
          nameservers:
            addresses:
              - 8.8.8.8
    ```

    ```bash
    sudo netplan apply
    ```

2. **Configurar IP estática en la VM Attacker:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.30.3/29
          nameservers:
            addresses:
              - 192.168.30.2
          routes:
            - to: default
              via: 192.168.30.2
    ```

3. **Habilitar reenvío de paquetes en la VM Server:**

    ```bash
    sudo nano /etc/sysctl.conf
    # Asegurar: net.ipv4.ip_forward=1
    sudo sysctl -p
    sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

### Ejercicio 2: Instalación y Configuración del Servidor Objetivo

1. **Instalar Nginx y SSH:**

    ```bash
    sudo apt update
    sudo apt install nginx openssh-server -y
    sudo systemctl enable --now nginx
    sudo systemctl enable --now sshd
    ```

2. **Crear un sitio web de prueba:**

    ```bash
    sudo bash -c 'echo "<h1>Servidor de Prueba - Lab 5.2</h1>" > /var/www/html/index.html'
    sudo bash -c 'echo "<p>Objetivo de simulación de intrusiones</p>" >> /var/www/html/index.html'
    ```

3. **Verificar servicios activos:**

    ```bash
    sudo systemctl status nginx
    sudo systemctl status sshd
    sudo ss -tulnp | grep -E "nginx|sshd"
    ```

### Ejercicio 3: Instalación y Configuración de Fail2ban

1. **Instalar Fail2ban:**

    ```bash
    sudo apt update
    sudo apt install fail2ban -y
    sudo systemctl enable --now fail2ban
    ```

2. **Crear configuración local de jails:**

    ```bash
    sudo nano /etc/fail2ban/jail.local
    ```

    ```ini
    [DEFAULT]
    # Tiempo de baneo en segundos (10 minutos)
    bantime = 600
    # Ventana de tiempo para contar intentos (10 minutos)
    findtime = 600
    # Número máximo de intentos fallidos
    maxretry = 3
    # Backend para detectar intentos
    backend = systemd

    [sshd]
    enabled = true
    port = ssh
    filter = sshd
    logpath = /var/log/auth.log
    maxretry = 3

    [nginx-http-auth]
    enabled = true
    port = http,https
    filter = nginx-http-auth
    logpath = /var/log/nginx/error.log
    maxretry = 3
    ```

    > **Explicación:**
    > - `bantime`: Duración del bloqueo de la IP.
    > - `findtime`: Ventana temporal para contar intentos fallidos.
    > - `maxretry`: Intentos permitidos antes del baneo.
    > - `[sshd]`: Protege contra fuerza bruta a SSH.
    > - `[nginx-http-auth]`: Protege contra intentos de autenticación básica HTTP.

3. **Reiniciar Fail2ban y verificar estado:**

    ```bash
    sudo systemctl restart fail2ban
    sudo fail2ban-client status
    sudo fail2ban-client status sshd
    ```

### Ejercicio 4: Simulación de Ataque de Fuerza Bruta a SSH

En esta sección, la VM `Attacker` simulará un ataque automatizado contra el SSH del `Server`.

1. **En la VM Attacker, instalar Hydra:**

    ```bash
    sudo apt update
    sudo apt install hydra -y
    ```

2. **Crear un archivo con contraseñas comunes para el ataque:**

    ```bash
    nano passwords.txt
    ```

    ```
    123456
    password
    admin
    ubuntu
    root
    12345678
    qwerty
    letmein
    welcome
    princess
    ```

3. **Ejecutar el ataque de fuerza bruta contra SSH:**

    ```bash
    hydra -l usuario -P passwords.txt ssh://192.168.30.2
    ```

    > **Nota:** Reemplaza `usuario` por un nombre de usuario válido en el servidor (por ejemplo, el usuario que creaste al instalar Ubuntu). Si el usuario no existe, el ataque seguirá generando logs de "Failed password".

    > **Advertencia:** Este ataque se realiza únicamente en el entorno de laboratorio aislado. Nunca ejecutes ataques de fuerza bruta sin autorización explícita.

4. **En la VM Server, verificar los intentos fallidos en `auth.log`:**

    ```bash
    sudo grep "Failed password" /var/log/auth.log
    ```

    Contar el número de intentos fallidos:

    ```bash
    sudo grep "Failed password" /var/log/auth.log | wc -l
    ```

    Ver las IPs que intentaron acceder:

    ```bash
    sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
    ```

5. **Verificar que Fail2ban haya baneado la IP atacante:**

    ```bash
    sudo fail2ban-client status sshd
    ```

    Ver la lista de IPs baneadas:

    ```bash
    sudo fail2ban-client status sshd | grep "Banned IP list"
    ```

    Verificar la regla de firewall creada por Fail2ban:

    ```bash
    sudo iptables -L -n | grep fail2ban
    ```

6. **Intentar conectar desde la VM Attacker después del baneo:**

    ```bash
    ssh usuario@192.168.30.2
    ```

    > **Resultado esperado:** La conexión debe fallar o quedarse colgada porque la IP `192.168.30.3` está bloqueada por el firewall.

7. **Desbanear la IP (opcional, para continuar con prácticas):**

    ```bash
    sudo fail2ban-client set sshd unbanip 192.168.30.3
    ```

### Ejercicio 5: Simulación de Escaneo y Ataque a Servicio Web

1. **En la VM Attacker, instalar Nmap:**

    ```bash
    sudo apt install nmap -y
    ```

2. **Realizar un escaneo de puertos contra el Server:**

    ```bash
    nmap -sV 192.168.30.2
    ```

    > **Resultado esperado:** Debería mostrar puertos 22 (SSH) y 80 (HTTP) abiertos.

3. **Simular un reconocimiento web (peticiones a rutas inexistentes):**

    ```bash
    for i in $(seq 1 50); do
      curl -s -o /dev/null -w "%{http_code}" http://192.168.30.2/admin$i
      echo " -> /admin$i"
    done
    ```

4. **En la VM Server, analizar `access.log` de Nginx:**

    Ver códigos 404 (rutas inexistentes):

    ```bash
    sudo grep " 404 " /var/log/nginx/access.log | head -20
    ```

    Contar cuántos 404 generó cada IP:

    ```bash
    sudo grep " 404 " /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -nr
    ```

    Buscar patrones de escaneo (múltiples peticiones seguidas):

    ```bash
    sudo awk '{print $1, $4, $6, $7, $9}' /var/log/nginx/access.log | grep " 404 " | head -30
    ```

    > **Análisis:** Una IP que genera decenas o cientos de códigos 404 en pocos segundos es un indicador claro de reconocimiento o escaneo automatizado.

5. **Verificar `error.log` por intentos de autenticación HTTP:**

    ```bash
    sudo grep "auth" /var/log/nginx/error.log | head -20
    ```

### Ejercicio 6: Ciclo de Respuesta a Incidentes

Documenta el siguiente proceso como si fueras el analista de seguridad respondiendo al incidente.

1. **Identificar:**

    - ¿Qué servicio está siendo atacado?
    - ¿Desde qué IP(s) proviene el ataque?
    - ¿Es un ataque de fuerza bruta, escaneo o ambos?

    Comandos de ayuda:

    ```bash
    sudo lastb  # Muestra intentos de login fallidos
    sudo grep "Failed password" /var/log/auth.log | tail -20
    sudo grep " 404 " /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
    ```

2. **Contener:**

    - Fail2ban ya aplicó una contención automática bloqueando la IP.
    - Si el ataque proviene de una IP nueva, bánala manualmente:

    ```bash
    sudo ufw deny from 192.168.30.3
    ```

3. **Erradicar:**

    - Verificar que no haya habido accesos exitosos previos:

    ```bash
    sudo grep "Accepted" /var/log/auth.log
    ```

    - Si encontraras un acceso exitoso desde la IP atacante, cambiarías las credenciales de inmediato:

    ```bash
    sudo passwd usuario
    ```

4. **Recuperar:**

    - Verificar que los servicios legítimos siguen funcionando:

    ```bash
    sudo systemctl status sshd
    sudo systemctl status nginx
    sudo systemctl status fail2ban
    ```

    - Desbanear la IP una vez confirmado que el ataque ha cesado:

    ```bash
    sudo fail2ban-client set sshd unbanip 192.168.30.3
    ```

5. **Documentar:**

    - Registrar en un archivo de incidente:
        - Fecha y hora del ataque.
        - IP atacante.
        - Servicio afectado.
        - Número de intentos fallidos.
        - Acciones tomadas (bloqueo, verificación de accesos).
        - Estado final (resuelto, en monitoreo).

    Ejemplo de plantilla:

    ```markdown
    ## Incidente #001 - Lab 5.2

    - **Fecha:** [Fecha del laboratorio]
    - **Hora de detección:** [Hora]
    - **IP Atacante:** 192.168.30.3
    - **Servicio Afectado:** SSH (sshd)
    - **Tipo de Ataque:** Fuerza bruta
    - **Intentos Fallidos:** 15
    - **Accesos Exitosos:** Ninguno detectado
    - **Acciones Tomadas:**
      - Verificación de logs en `/var/log/auth.log`.
      - Confirmación de bloqueo automático por Fail2ban.
      - Verificación de estado de servicios.
    - **Estado:** Resuelto
    ```

## ⚙️ Sección 3: Práctica en Grupo (Red entre Pares)

El objetivo es que cada grupo de **2 estudiantes** simule un escenario completo de ataque y defensa en la red del laboratorio.

### 1. Roles y Asignación por Grupo

| Integrante | Rol | VM a Configurar | Requisitos de Red |
| - | - | - | - |
| **Integrante 1** | **Defensor** | VM con Nginx, SSH, Fail2ban | Adaptador en modo **Puente (Bridge)** |
| **Integrante 2** | **Atacante** | VM con Hydra, Nmap, curl | Adaptador en modo **Puente (Bridge)** |

### 2. Tareas del Integrante 1 (Defensor)

1. **Configurar la VM en modo Puente** y asignar una IP estática dentro del rango de la red del laboratorio.

2. **Instalar Nginx, SSH y Fail2ban** como en la práctica individual.

3. **Configurar Fail2ban** con las siguientes reglas mínimas:
    - SSH: máximo 3 intentos fallidos en 5 minutos.
    - Baneo de 10 minutos.

4. **Crear un sitio web** con una página de bienvenida que incluya el nombre del grupo.

5. **Asegurar el servidor** aplicando al menos:
    - UFW con política de denegar por defecto.
    - Permitir solo puertos 22, 80 y 443.

6. **No revelar** al atacante qué contraseñas o usuarios existen.

### 3. Tareas del Integrante 2 (Atacante)

1. **Configurar la VM en modo Puente** y asignar una IP estática.

2. **Realizar reconocimiento:**
    - Escaneo de puertos con Nmap contra la IP del defensor.
    - Identificar servicios activos y versiones.

3. **Simular ataque de fuerza bruta:**
    - Ejecutar Hydra contra el SSH del defensor usando un diccionario de contraseñas.
    - Intentar al menos 20 contraseñas diferentes.

4. **Simular reconocimiento web:**
    - Generar múltiples peticiones 404 contra el servidor web.

### 4. Fase de Detección y Respuesta (ambos integrantes)

1. **El defensor debe detectar el ataque** analizando:
    - `auth.log` para intentos SSH fallidos.
    - `access.log` para peticiones web anómalas.
    - Estado de Fail2ban (`fail2ban-client status`).

2. **El defensor debe responder:**
    - Confirmar que la IP del atacante fue baneada.
    - Verificar que no hubo accesos exitosos.
    - Aplicar contención adicional si es necesario (UFW manual).

3. **Intercambio de roles:**
    - Una vez finalizado el primer round, los integrantes **intercambian roles**.
    - El atacante ahora defiende y el defensor ahora ataca.

### 5. Pruebas Cruzadas y Verificación

Después de ambos rounds, cada grupo debe verificar:

1. **¿Fail2ban bloqueó correctamente las IPs atacantes en ambos escenarios?**

    ```bash
    sudo fail2ban-client status sshd
    ```

2. **¿Los logs muestran patrones claros de ataque?**

    ```bash
    sudo grep "Failed password" /var/log/auth.log | wc -l
    sudo grep " 404 " /var/log/nginx/access.log | wc -l
    ```

3. **¿Los servicios legítimos siguen accesibles después del ataque?**

    ```bash
    curl http://IP_DEL_COMPAÑERO
    ssh usuario@IP_DEL_COMPAÑERO
    ```

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual (Fail2ban, simulación, análisis de logs): 35 pts
- Desarrollo de la práctica grupal (ataque/defensa, intercambio de roles): 35 pts
- Informe detallado con capturas de pantalla, análisis de logs y documentación del incidente: 30 pts
