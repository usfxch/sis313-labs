# Laboratorio 4.2: Servidor DNS Primario e Integración con Servicio Web

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Configurar un servidor DNS primario (maestro)** usando BIND9 para una red interna.

- **Crear y administrar una zona DNS directa** con registros SOA, NS, A y CNAME.

- **Verificar la resolución de nombres** mediante herramientas como `dig` y `nslookup`.

- **Integrar el DNS con un servicio web (Nginx)**, permitiendo el acceso al sitio por nombre de dominio en lugar de dirección IP.

- **Comprender la segmentación de red** y el rol del DNS en la infraestructura de servicios.

## 🛠️ Sección 1: Preparación del Entorno Virtual (Práctica Individual)

El entorno se desarrollará en una sola PC utilizando **3 Máquinas Virtuales (VMs)** con **Ubuntu Server 24.04 LTS**, simulando una red interna con resolución de nombres propia.

1. **Arquitectura de Red y Asignación de IPs**

    La red interna utilizará el segmento `192.168.10.0/29`.

    | VM | Hostname | Rol | Interfaces y Conexión | IP Interna (`/29`) |
    | - | - | - | - | - |
    | `Lab4.2-DNS` | `dns` | **SERVIDOR DNS (BIND9)** | NAT (Internet) + Red Interna | `192.168.10.2`|
    | `Lab4.2-Web` | `web` | **SERVIDOR WEB (Nginx)** | Red Interna | `192.168.10.3` |
    | `Lab4.2-Client` | `client` | **CLIENTE DE PRUEBAS** | Red Interna | `192.168.10.4` |

    - **Gateway (GW):** La interfaz interna de la VM `Lab4.2-DNS` actuará como puerta de enlace para el tráfico de salida de las VMs `Web` y `Client`. (Se configura a mano el *binding* y el reenvío de paquetes).

2. **Configuración de Red en VirtualBox**

    1. **Crear la Red Interna:** En VirtualBox, ir a **Herramientas → Redes → Crear**. Nombrar la red, ej., `Red_Lab4_2`.

    2. **Configurar Interfaces de las VMs:**

        - **VM Lab4.2-DNS (Servidor DNS):**

            - Adaptador 1: **NAT** (Acceso a Internet).

            - Adaptador 2: **Red Interna** (`Red_Lab4_2`).

        - **VM Lab4.2-Web (Servidor Web):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_2`).

        - **VM Lab4.2-Client (Cliente):**

            - Adaptador 1: **Red Interna** (`Red_Lab4_2`).
    
    3. **Reenvío de Puertos (Port Forwarding) en VM Lab4.2-DNS (NAT)**

        Configurar en el Adaptador 1 (NAT) de la VM `Lab4.2-DNS` para acceso desde la PC anfitriona.

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **SSH** | TCP | 127.0.0.1 | **2222** | 10.0.2.15 | 22 | Acceso Remoto |
        | **DNS** | TCP/UDP | 127.0.0.1 | **5353** | 10.0.2.15 | 53 | Consultas DNS |

    4. **Reenvío de Puertos (Port Forwarding) para acceso Web**

        Para poder acceder al sitio web desde el navegador del anfitrión usando el nombre de dominio, también se puede configurar un reenvío directo a la VM Web (opcional, o acceder vía el cliente).

        | Nombre | Protocolo | IP Host | Puerto Host | IP Invitado | Puerto Invitado | Propósito |
        | - | - | - | - | - | - | - |
        | **WEB** | TCP | 127.0.0.1 | **8080** | 192.168.10.3 | 80 | Acceso Web (Nginx) |

        > **Nota:** Para que el nombre `www.lab42.local` funcione en el anfitrión, deberás editar el archivo `hosts` de tu sistema operativo anfitrión apuntando `www.lab42.local` a `127.0.0.1` y acceder por el puerto 8080, o configurar el DNS del anfitrión para que use `127.0.0.1:5353`.

## 💻 Sección 2: Práctica Guiada (Ejercicios Individuales)

### Ejercicio 1: Configuración de Red Estática (en las 3 VMs)

1. **Configurar IP estática en la VM DNS:**

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
            - 192.168.10.2/29
          nameservers:
            addresses:
              - 8.8.8.8
    ```

    Aplica los cambios:

    ```bash
    sudo netplan apply
    ```

2. **Configurar IP estática en la VM Web:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.10.3/29
          nameservers:
            addresses:
              - 192.168.10.2
          routes:
            - to: default
              via: 192.168.10.2
    ```

    > **Explicación:** El servidor DNS (`192.168.10.2`) también actúa como gateway. El `nameservers` apunta al DNS interno.

3. **Configurar IP estática en la VM Cliente:**

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s8:
          dhcp4: no
          optional: true
          addresses:
            - 192.168.10.4/29
          nameservers:
            addresses:
              - 192.168.10.2
          routes:
            - to: default
              via: 192.168.10.2
    ```

4. **Habilitar reenvío de paquetes en la VM DNS:**

    La VM DNS debe enrutar el tráfico de la red interna hacia Internet (NAT).

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

### Ejercicio 2: Instalación y Configuración de BIND9 (VM Lab4.2-DNS)

1. **Instalar BIND9:**

    ```bash
    sudo apt update
    sudo apt install bind9 bind9utils bind9-doc -y
    ```

2. **Configurar la zona en `named.conf.local`:**

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Agrega el siguiente bloque:

    ```bind
    zone "lab42.local" {
        type master;
        file "/etc/bind/db.lab42.local";
    };
    ```

    > **Explicación:** Definimos una zona maestra (primaria) para el dominio `lab42.local`. El archivo `db.lab42.local` contendrá los registros DNS.

3. **Crear el archivo de zona:**

    Es recomendable copiar el archivo de plantilla incluido en BIND:

    ```bash
    sudo cp /etc/bind/db.local /etc/bind/db.lab42.local
    sudo nano /etc/bind/db.lab42.local
    ```

    Reemplaza su contenido por:

    ```bind
    $TTL    604800
    @       IN      SOA     dns.lab42.local. admin.lab42.local. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      dns.lab42.local.
    dns     IN      A       192.168.10.2
    www     IN      A       192.168.10.3
    web     IN      CNAME   www.lab42.local.
    ```

    > **Explicación de registros:**
    > - **SOA:** Inicio de Autoridad. Define el servidor primario y parámetros de la zona.
    > - **NS:** Servidor de nombres autoritativo para la zona.
    > - **A (dns):** El nombre `dns.lab42.local` resuelve a `192.168.10.2`.
    > - **A (www):** El nombre `www.lab42.local` resuelve a `192.168.10.3` (VM Web).
    > - **CNAME (web):** El alias `web.lab42.local` apunta a `www.lab42.local`.

4. **Verificar la configuración:**

    ```bash
    sudo named-checkconf
    sudo named-checkzone lab42.local /etc/bind/db.lab42.local
    ```

    Deberías ver un mensaje similar a:

    ```
    zone lab42.local/IN: loaded serial 2
    OK
    ```

5. **Reiniciar BIND9 y verificar estado:**

    ```bash
    sudo systemctl restart bind9
    sudo systemctl status bind9
    ```

    Verifica que esté escuchando en el puerto 53:

    ```bash
    sudo netstat -tulnp | grep named
    ```

### Ejercicio 3: Configuración del Servidor Web (VM Lab4.2-Web)

1. **Instalar Nginx:**

    ```bash
    sudo apt update
    sudo apt install nginx -y
    sudo systemctl enable --now nginx
    ```

2. **Configurar Virtual Host para el dominio:**

    Crea un nuevo bloque de servidor:

    ```bash
    sudo nano /etc/nginx/sites-available/lab42.local
    ```

    ```nginx
    server {
        listen 80;
        server_name www.lab42.local lab42.local;

        root /var/www/lab42.local;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

3. **Activar el sitio y crear el contenido:**

    ```bash
    sudo mkdir -p /var/www/lab42.local
    sudo ln -s /etc/nginx/sites-available/lab42.local /etc/nginx/sites-enabled/
    sudo rm /etc/nginx/sites-enabled/default
    ```

    Crea una página de prueba:

    ```bash
    sudo bash -c 'echo "<h1>Bienvenido a www.lab42.local</h1>" > /var/www/lab42.local/index.html'
    sudo bash -c 'echo "<p>Servido desde la VM Web (192.168.10.3)</p>" >> /var/www/lab42.local/index.html'
    ```

    Verifica la sintaxis de Nginx y reinicia:

    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

### Ejercicio 4: Pruebas de Resolución y Acceso

1. **Desde la VM Cliente:**

    Asegúrate de que el cliente esté usando el DNS interno (ya configurado en Netplan con `nameservers: addresses: [192.168.10.2]`).

    Prueba la resolución con `dig`:

    ```bash
    dig @192.168.10.2 www.lab42.local
    ```

    Prueba con `nslookup`:

    ```bash
    nslookup www.lab42.local 192.168.10.2
    ```

    Prueba el alias CNAME:

    ```bash
    dig @192.168.10.2 web.lab42.local
    ```

    Accede al sitio web por nombre:

    ```bash
    curl http://www.lab42.local
    ```

    > **Resultado esperado:** Deberías ver el HTML de bienvenida servido desde la VM Web.

2. **Desde la PC Anfitriona:**

    Para que tu computadora anfitriona resuelva el dominio, edita el archivo `hosts`:

    - **Linux/macOS:** `sudo nano /etc/hosts`
    - **Windows:** `C:\Windows\System32\drivers\etc\hosts`

    Agrega la siguiente línea (usando el port forwarding de la VM Web):

    ```
    127.0.0.1   www.lab42.local
    127.0.0.1   lab42.local
    ```

    Luego accede desde tu navegador a:

    ```
    http://127.0.0.1:8080
    ```

    > **Nota:** Como el port forwarding apunta directamente al puerto 80 de la VM Web, el navegador mostrará el sitio sin necesidad de DNS. Si deseas probar el DNS completo desde el anfitrión, configura tu sistema para usar `127.0.0.1:5353` como servidor DNS (avanzado) o usa la VM Cliente como referencia de prueba.

## ⚙️ Sección 3: Práctica en Grupo (Red entre Pares)

El objetivo es que cada grupo de **2 estudiantes** despliegue una infraestructura DNS + Web funcional en la red del laboratorio, de modo que ambas PCs anfitrionas puedan resolver nombres y acceder al sitio.

### 1. Roles y Asignación por Grupo

Cada grupo está compuesto por 2 integrantes. Cada uno configurará al menos una VM en su PC anfitriona.

| Integrante | Rol Principal | VM a Configurar | Requisitos de Red |
| - | - | - | - |
| **Integrante 1** | **Servidor DNS** | VM con BIND9 | Adaptador en modo **Puente (Bridge)** |
| **Integrante 2** | **Servidor Web** | VM con Nginx (sitio estático HTML) | Adaptador en modo **Puente (Bridge)** |

> **Nota:** El modo **Puente (Bridge)** permite que la VM obtenga una IP directamente de la red física del laboratorio (o use una IP estática en el rango de la red del aula), haciendo visible la VM para las demás PCs.

### 2. Dominio Local del Grupo

Cada grupo deberá elegir su propio dominio local con la extensión TLD que prefieran. Ejemplos:

- `grupo1.red`
- `equipo-a.local`
- `sis313.corp`
- `los-pepes.red`

> **Restricción:** El dominio debe ser único dentro del aula para evitar colisiones. El docente validará que no haya duplicados.

### 3. Tareas del Integrante 1 (Servidor DNS)

1. **Configurar la VM DNS en modo Puente** y asignar una IP estática dentro del rango de la red del laboratorio (consulta al docente el rango disponible).

2. **Instalar y configurar BIND9** como en la práctica individual, pero usando el **dominio del grupo**.

3. **Configurar la Zona Directa:** Crear registros SOA, NS, A y CNAME para el dominio elegido. Ejemplo para `los-pepes.red`:

    ```bind
    $TTL    604800
    @       IN      SOA     dns.los-pepes.red. admin.los-pepes.red. (
                                  1         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      dns.los-pepes.red.
    dns     IN      A       192.168.5.101
    www     IN      A       192.168.5.102
    web     IN      CNAME   www.los-pepes.red.
    ```

    > **Nota:** Reemplaza las IPs por las correspondientes a las VMs de tu grupo en la red del aula. El registro **CNAME** crea un alias para que `web.los-pepes.red` también apunte al servidor web.

4. **Configurar la Zona Inversa:** Crear la zona inversa correspondiente al segmento de red asignado. Ejemplo para el rango `192.168.5.0/24`:

    En `/etc/bind/named.conf.local`, agrega:

    ```bind
    zone "5.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.192.168.5";
    };
    ```

    Crea el archivo `/etc/bind/db.192.168.5`:

    ```bind
    $TTL    604800
    @       IN      SOA     dns.los-pepes.red. admin.los-pepes.red. (
                                  1         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      dns.los-pepes.red.
    101     IN      PTR     dns.los-pepes.red.
    102     IN      PTR     www.los-pepes.red.
    ```

    > **Explicación:** La zona inversa permite obtener el nombre de dominio a partir de una dirección IP. El archivo se lee de derecha a izquierda omitiendo el último octeto.

5. **Verificar la configuración:**

    ```bash
    sudo named-checkconf
    sudo named-checkzone los-pepes.red /etc/bind/db.los-pepes.red
    sudo named-checkzone 5.168.192.in-addr.arpa /etc/bind/db.192.168.5
    sudo systemctl restart bind9
    ```

### 4. Tareas del Integrante 2 (Servidor Web)

1. **Configurar la VM Web en modo Puente** y asignar una IP estática dentro del rango de la red del laboratorio.

2. **Instalar Nginx** y configurar el sitio para responder al dominio del grupo.

3. **Crear un sitio web estático en HTML** (no PHP, no Node.js). Debe incluir al menos:
    - Un título que identifique al grupo.
    - El nombre de los integrantes.
    - Una imagen o texto de bienvenida.

    Ejemplo de `/var/www/los-pepes.red/index.html`:

    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Los Pepes - SIS313</title>
    </head>
    <body>
        <h1>Bienvenidos a los-pepes.red</h1>
        <p>Integrantes: Juan Perez, Maria Lopez</p>
        <p>Servidor Web de prueba para el Laboratorio 4.2</p>
    </body>
    </html>
    ```

4. **Configurar el Virtual Host:**

    ```nginx
    server {
        listen 80;
        server_name www.los-pepes.red los-pepes.red;
        root /var/www/los-pepes.red;
        index index.html;
    }
    ```

### 5. Pruebas desde las PCs Anfitrionas (Clientes)

Ambas PCs anfitrionas actuarán como clientes de la red. Deberán configurar su **DNS principal** para que apunte a la IP de la VM DNS del Integrante 1.

- **Windows:** Panel de Control → Red → IPv4 → DNS preferido: `IP_DNS_VM`
- **Linux:** Modificar `/etc/resolv.conf` o la configuración de red gráfica.
- **macOS:** Preferencias del Sistema → Red → DNS.

> **Importante:** Asegúrate de que el firewall de la VM DNS (UFW) permita conexiones al puerto 53 desde la red del aula:
> ```bash
> sudo ufw allow from 192.168.5.0/24 to any port 53
> ```

**Pruebas a realizar:**

1. **Resolución directa (Zona Directa):**
    ```bash
    nslookup www.los-pepes.red
    dig www.los-pepes.red
    ```

2. **Resolución inversa (Zona Inversa):**
    ```bash
    nslookup IP_DEL_SERVIDOR_WEB
    dig -x IP_DEL_SERVIDOR_WEB
    ```

    > **Resultado esperado:** La resolución inversa debe devolver el nombre `www.los-pepes.red`.

3. **Acceso Web por nombre:**
    Abre un navegador en la PC anfitriona y accede a:
    ```
    http://www.los-pepes.red
    ```

    > **Resultado esperado:** Deberías ver la página HTML estática creada por el Integrante 2.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos:

- Desarrollo de la práctica individual: 35 pts 
- Desarrollo de la práctica grupal: 35 pts
- Informe detallado en formato markdown con comandos utilizados y capturas de pantalla que demuestren ambas prácticas: 30 pts
