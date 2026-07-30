# Laboratorio 3.1: Proxy Inverso con Balanceador de Carga Avanzado y Servidores Web NGINX

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Comprender y configurar un proxy inverso y balanceador de carga** con **NGINX**.

- **Diseñar y configurar una red interna** con una máscara de subred `/29` y compartir acceso a internet desde el proxy.

- **Configurar NGINX como proxy inverso** en los servidores backend para servir aplicaciones en PHP o Node.js.

- **Implementar y comparar** los algoritmos de balanceo de carga **round robin**, **least connection** e **IP hash**.

- **Analizar y demostrar el funcionamiento** del balanceo de carga en escenarios reales, incluyendo la simulación de la caída de servidores.

### 📋 Tabla Resumen de Infraestructura

| Máquina Virtual | Rol | Imagen SO | Interfaz | Red | IP |
|---|---|---|---|---|---|
| Lab3_1-Proxy | Proxy Inverso + Balanceador | Ubuntu 24.04 | 1: enp0s3<br>2: enp0s8 | 1: NAT (DHCP)<br> 2: Red Interna | 1: 10.0.2.15<br>2: 192.168.10.1 |
| Lab3_1-WebServer1 | Servidor Web | Alpine Linux 3.22 | eth0 | Red Interna | 192.168.10.2 |
| Lab3_1-WebServer2 | Servidor Web | Alpine Linux 3.22 | eth0 | Red Interna | 192.168.10.3 |

## 🛠️ Sección 1: Preparación del Entorno Virtual

En esta sección, se configurará la infraestructura en tu software de virtualización. Se utilizarán tres máquinas virtuales.

1. **Máquina Virtual 1: Servidor Proxy Inverso y Balanceador de Carga**

    Crear una máquina virtual en VirtualBox con la siguiente configuración:

    - **Nombre e imagen a utilizar:**
        - **Nombre:** Lab3.1-Proxy
        - **Imagen ISO:** ubuntu-24.04.3-live-server-amd64.iso

    - **Hardware:**
        - **Memoria base:** 2048 MB
        - **Procesadores:** 1 CPU

    - **Disco duro:**
        - **Capacidad:** 10,00 GB 

    - **Red:** Configuración de Red:
        - **Adaptador 1:** Habilitar adaptador de red:
            - **Conectado a:** NAT (Para acceder Internet)
        - **Adaptador 2:** Habilitar adaptador de red:
            - **Conectado a:** Red Interna (Para conectarse a los servidores Web Backend y compartir Internet con ellos)
            - **Nombre:** Lab3.1-SW (Acturará como Switch dentro de la Red Interna)

2. **Máquina Virtual 2: Servidor Web 1**

    Crear una máquina virtual en VirtualBox con la siguiente configuración:

    - **Nombre e imagen a utilizar:**
        - **Nombre:** Lab3.1-WebServer1
        - **Imagen ISO:** alpine-standard-3.22.1-x86_64.iso ([Alpine Linux](https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-standard-3.22.1-x86_64.iso))
        - **Tipo:** Linux
        - **Subtipo:** Other Linux
        - **Versión:** Other Linux (64-bit)

    - **Hardware:**
        - **Memoria base:** 1024 MB
        - **Procesadores:** 1 CPU

    - **Disco duro:**
        - **Capacidad:** 6,00 GB 

    - **Red:** Configuración de Red:
        - **Adaptador 1:** Habilitar adaptador de red:
            - **Conectado a:** NAT (Inicialmente para acceder Internet. Una vez instalado, se debe modificar a "Red Interna" para conectar a la red interna).

        - **Reenvío de puertos:** Añadir las siguientes reglas de reenvío:
        
            | Nombre | Protocolo | IP anfitrión | Puerto anfitrión | IP invitado | Puerto invitado |
            | - | - | - | - | - | - |
            | SSH | TCP |   | 2222 | 10.0.2.15 | 22 |
            | HTTP | TCP |   | 8080 | 10.0.2.15 | 80 |

3. **Máquina Virtual 3: Servidor Web 2**

   Crear una máquina virtual en VirtualBox con la siguiente configuración:

    - **Nombre e imagen a utilizar:**
        - **Nombre:** Lab3.1-WebServer2
        - **Imagen ISO:** alpine-standard-3.22.1-x86_64.iso ([Alpine Linux](https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-standard-3.22.1-x86_64.iso))
        - **Tipo:** Linux
        - **Subtipo:** Other Linux
        - **Versión:** Other Linux (64-bit)

    - **Hardware:**
        - **Memoria base:** 1024 MB
        - **Procesadores:** 1 CPU

    - **Disco duro:**
        - **Capacidad:** 6,00 GB 

    - **Red:** Configuración de Red:
        - **Adaptador 1:** Habilitar adaptador de red:
            - **Conectado a:** NAT (Temporal, solo para descargar paquetes necesarios para instalar el SO. Una vez instalado, se debe modificar a "Red Interna").

## 💻 Sección 2: Práctica guiada

### Paso 1: Instalación y configuración de red de Ubuntu Server 24.04 (Servidor Proxy y Balanceador de Carga)

Seguir los pasos realizados en anteriores laboratorios a diferencia de la configuración de red, `Network configuration`, donde debes:

1. Dejar tal cual la configuración de la interfaz de red `enp0s3` con `DHCP`, es muy probable que la IP asignada sea la `10.0.2.15/24`.

2. Configurar la interfaz `enp0s8` con `IPv4`:
    - Método de IPv4: Manual
    - Subred: 192.168.10.0/29
    - Dirección: 192.168.10.1
    - Puerta de enlace: (vacío)
    - Servidores de nombres: (vacío)
    - Dominios de búsqueda: (vacío)

### Paso 2: Instalación y configuración de red de Alpine Linux (Servidor Web 1)

Iniciar la máquina virtual y realizar las siguientes tareas:
1. Una vez iniciado el S.O., debes iniciar sesión con usuario root:
    <pre>
    (...)
    localhost login: <strong>root</strong> ⮐
</pre>

2. Ejecutar el instalador de Alpine Linux:
    <pre>
    localhost:~# <strong>setup-alpine</strong> ⮐
</pre>

3. Seleccione la disposición del teclado:
    <pre>
    Select keyboard layout: [none] <strong>us</strong> ⮐ 
</pre>

4. Seleccione la variante del teclado:
    <pre>
    Select variant (or 'abort'): <strong>us</strong> ⮐
</pre>

5. Introduzca el hostname de la máquina virtual:
    <pre>
    Hostname
    --------
    Enter system hostname (fully qualified form, e.g. 'foo.example.org') [localhost] <strong>webserver1</strong> ⮐
</pre>

6. Seleccione la interfaz que tendrá que configurar:
    <pre>
    Interface
    ---------
    (...)
    Which one do you want to initialize? (or '?' or 'done') [eth0] ⮐
    
    IP address for eth0? (or 'dhcp', 'none' ?) [dhcp] ⮐ 

    Do you want to do any manual network configuration? (y/n) [n] ⮐ 
</pre>

7. Introduce la contraseña del usuario root:
    <pre>
    Root password
    -------------
    New password: <strong>******</strong> ⮐
    Retype password: <strong>******</strong> ⮐
</pre>

8. Selecciona la zona horaria:
    <pre>
    Timezone
    --------
    (...)
    Which timezone are you in? (or '?' or 'none') [UTC] <strong>America/La_Paz</strong> ⮐
</pre>

9. Configuración del Proxy (Internet):
    <pre>
    Proxy
    -----
    HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] ⮐
</pre>

10. Configuración del servidor NTP:
    <pre>
    Network Time Protocol
    ---------------------
    (...)
    Which NTP client to run? ('busybox', 'openntp', 'chrony' or 'none') [busybox] ⮐
    </pre>

11. Configuración de los repositorios de paquetes de Alpine Linux:
    <pre>
    APK Mirror
    ----------
    (...)
    Enter mirror number or URL: [1] ⮐
    </pre>

12. Configuración del usuario alternativo a root:
    <pre>
     User 
    ------
    Setup a user? (enter a lower-case loginname, or 'no') [no] <strong>marcelo</strong> ⮐
    (...)
    Full name for user marcelo [marcelo] <strong>Marcelo Quispe Ortega</strong> ⮐
    (...)
    New password: <strong>******</strong> ⮐
    Retype password: <strong>******</strong> ⮐
    (...)
    Enter ssh key or URL for marcelo (or 'none') [none] ⮐
    (...)
    Which ssh server? ('openssh', 'dropbear' or 'none') [openssh] ⮐
    </pre>

13. Instalación en Disco duro:
    <pre>
    Disk & Install
    --------------
    Which disk(s) would you like to use? (or '?' for help or 'none') [none] <strong>sda</strong> ⮐
    (...)
    How would you like to use it? ('sys', 'data', 'crypt', 'lvm' or '?' for help) [?] <strong>sys</strong> ⮐
    (...)
    WARNING: Erase the above disk(s) and continue? (y/n) [n] <strong>y</strong> ⮐
    (...)
    Installation is complete. Please reboot.
    webserver1:~# <strong>poweroff</strong> ⮐
    </pre>

14. Una vez apagada la máquina virtual, la seleccionamos y hacemos clic en `Configuración`, luego en `Almacenamiento`, desmontamos del `IDE Secundario` (CD/DVD) la imagen del ISO de `Alpine Linux` y guardamos la configuración.

15. Iniciamos nuevamente la máquina virtual, iniciamos sesión como `root` e instalamos el paquetes `nano`:
    <pre>
    webserver1:~# <strong>apk add nano</strong> ⮐
    </pre>

    Apagamos la máquina virtual:
    <pre>
    webserver1:~# <strong>poweroff</strong> ⮐
    </pre>

16. Accedemos a VirtualBox, seleccionamos la máquina virtual `Lab3.1-WebServer1` y entramos en `Configuración` → `Almacenamiento`. Desmontamos la imagen ISO de Alpine Linux del `IDE Secundario` (CD/DVD) y guardamos la configuración.

17. Configuramos la máquina virtual para cambiar el `Adaptador 1` de Red a:
    - Conectado a: `Red Interna`
    - Nombre: `Lab3.1-SW`

18. Iniciamos nuevamente la máquina virtual, accedemos como `root` y configuramos nuevamente la interfaz de red:
    <pre>
    webserver1:~# <strong>nano /etc/network/interfaces</strong> ⮐
    </pre>
    
    ```
    auto eth0
    iface eth0 inet static
        address 192.168.10.2
        netmask 255.255.255.248
        gateway 192.168.10.1
    ```

    Reiniciamos el servicio de red:
    <pre>
    webserver1:~# <strong>/etc/init.d/networking restart</strong>
    </pre>

    Probamos la conexión con el `gateway` haciendo `ping`.
    <pre>
    webserver1:~# <strong>ping 192.168.10.1</strong>
    </pre>

### Paso 3: Instalación y configuración de red de Alpine Linux (Servidor Web 2)

Realizar las mismas tareas en el Paso 2.

### Paso 4: Compartir Acceso a Internet desde el Proxy

Para que los servidores backend puedan descargar e instalar paquetes, el servidor proxy debe actuar como un enrutador NAT.

- **En el servidor Ubuntu (Proxy):**

    - Habilita el reenvío de paquetes (IP forwarding) en el kernel.

        ```bash
        sudo sysctl -w net.ipv4.ip_forward=1
        ```
    
    - Configura las reglas de `iptables` para redirigir el tráfico de la red interna (`enp0s8`) a través de la interfaz que tiene acceso a internet (`enp0s3`).

        ```bash
        sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
        ```

        <!-- ```bash -->
        <!-- sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT -->
        <!-- ``` -->

        <!-- ```bash -->
        <!-- sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT -->
        <!-- ``` -->

    - Guarda las reglas para que persistan después de reiniciar el servidor.

        ```bash
        sudo apt install iptables-persistent
        ```

        ```bash
        sudo netfilter-persistent save
        ```

- **En los servidores Web 1 y Web 2:**
    - Configurar el servidor DNS para la resolución de dominios y salir a Internet.

        ```bash
        nano /etc/resolv.conf
        ```

        Modificar el contenido del archivo a:

        ```bash
        nameserver 8.8.8.8
        ```

        > **ℹ️ Nota importante sobre persistencia DNS en Alpine:** En Alpine Linux, `/etc/resolv.conf` puede ser sobrescrito. Para hacerlo persistente, edita `/etc/network/interfaces` directamente:
        > ```
        > auto eth0
        > iface eth0 inet static
        >     address 192.168.10.2
        >     netmask 255.255.255.248
        >     gateway 192.168.10.1
        >     dns-nameservers 8.8.8.8
        > ```
        > Luego: `/etc/init.d/networking restart`

### Paso 5: Configuración de los servidores web backend

- **Verifica la conexión a internet:** Desde los servidores Alpine, haz un `ping google.com` para confirmar que tienen acceso a internet.

- **Instala `nginx` y configura como servicio:**

    - Instala el paquete de `nginx`:

        ```bash
        apk add nginx
        ```

    - Configura el servicio para inicializarlo desde el arranque del sistema:

        ```bash
        rc-update add nginx default
        ```

    - Inicializa el servicio:

        ```bash
        rc-service nginx start 
        # Puedes utilizar también: /etc/init.d/nginx start
        ```

    - Por último, verifica si el servicio está funcionando:

        ```bash
        rc-service nginx status 
        # Puedes utilizar también: /etc/init.d/nginx status
        ```

- **OPCIÓN 1: Instala `php-fpm` y configura `nginx` como proxy inverso:**

    1. Habilita los repositorios de la Comunidad de Alpine Linux.

        ```bash
        nano /etc/apk/repositories
        ```

    2. Descomentar el repositorio de la Comunidad, debe verse de la siguiente manera:

        ```bash
        http://dl-cdn.alpinelinux.org/alpine/v3.22/main
        http://dl-cdn.alpinelinux.org/alpine/v3.22/community
        ```

    3. Actualiza la lista de los repositorios:

        ```bash
        apk update
        ```

    4. Instala los paquete `php-fpm`:

        ```bash
        apk add php php-fpm
        ```

    5. Verifica la versión de PHP que se instaló:
        ```bash
        php -v
        ```

    6. Habilita el servicio de `php-fpm` desde el arranque del sistema:
        ```bash
        rc-update add php-fpm83
        ```
    
    7. Inicia el servicio de `php-fpm`:
        ```bash
        rc-service php-fpm83 start
        ```

    8. Habilita el virtual host de `nginx` con `php-fpm`:

        - Configura el virtual host principal del servidor Web, para que `nginx` funcione como proxy inverso con `php-fpm`:

            ```bash
            nano /etc/nginx/http.d/default.conf
            ```

            ```nginx
            server {
                listen 80;
                server_name _;
                root /var/www/localhost/htdocs;

                location / {
                    index index.php index.html index.htm;
                    try_files $uri $uri/ =404;
                }

                location ~ \.php$ {
                    include fastcgi_params;
                    fastcgi_pass 127.0.0.1:9000;
                    fastcgi_index index.php;
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                }
            }
            ```

        - Configura la página de bienvenida del `php-fpm`:

            ```bash
            nano /var/www/localhost/htdocs/index.php
            ```

            ```html
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Hola Mundo desde el servidor <?=gethostname() ?> (<?=$_SERVER['SERVER_ADDR'] ?>)</title>
            </head>
            <body>
                <h1>¡Hola Mundo desde el servidor <?=gethostname() ?> (<?=$_SERVER['SERVER_ADDR'] ?>)!</h1>
            </body>
            </html>
            ```

        - Reinicia el servicio de `php-fpm`:

            ```bash
            /etc/init.d/php-fpm83 restart
            ```

        - Reinicia el servicio de `nginx`:

            ```bash
            /etc/init.d/nginx restart
            ```

    9. Prueba si el `webserver1` ya responde solicitudes:

        - Instala CURL para probar si funciona:
            ```bash
            apk add curl
            ```

        - Ejecuta una petición al servidor Web:
            ```bash
            curl http://localhost
            ```

        - Tu respuesta deberá ser la siguiente:
            ```
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Hola Mundo desde el servidor webserver1 (127.0.0.1)</title>
            </head>
            <body>
                <h1>¡Hola Mundo desde el servidor webserver1 (127.0.0.1)!</h1>
            </body>
            </html>
            ```

        **Repite los mismos pasos (del 1 al 9) con el otro servidor (webserver2).**

- **OPCIÓN 2: Instala Node.js y configura NGINX como Proxy Inverso:**

    Esta opción utiliza **Node.js** en lugar de PHP-FPM para servir la aplicación "Hola Mundo".

    1. Instala Node.js y npm:

        ```bash
        apk add nodejs npm
        ```

    2. Crea un directorio para la aplicación:

        ```bash
        mkdir -p /var/www/nodejs
        cd /var/www/nodejs
        ```

    3. Crea el archivo `package.json`:

        ```bash
        nano package.json
        ```

        ```json
        {
          "name": "hola-mundo-app",
          "version": "1.0.0",
          "description": "Aplicación simple Hola Mundo",
          "main": "index.js",
          "scripts": {
            "start": "node index.js"
          },
          "keywords": [],
          "author": "",
          "license": "ISC"
        }
        ```

    4. Crea el archivo `index.js` con la aplicación Node.js:

        ```bash
        nano index.js
        ```

        ```javascript
        const http = require('http');
        const os = require('os');
        const { execSync } = require('child_process');

        const hostname = os.hostname();
        const hostip = execSync('hostname -I').toString().trim().split(' ')[0];

        const server = http.createServer((req, res) => {
            const html = `
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Hola Mundo desde ${hostname} (${hostip})</title>
            </head>
            <body>
                <h1>¡Hola Mundo desde el servidor ${hostname} (${hostip})!</h1>
            </body>
            </html>
            `;
            res.statusCode = 200;
            res.setHeader('Content-Type', 'text/html; charset=utf-8');
            res.end(html);
        });

        const PORT = 3000;
        server.listen(PORT, '127.0.0.1', () => {
            console.log(`Servidor ejecutándose en http://127.0.0.1:${PORT}/`);\n        });
        ```

    5. Instala las dependencias (si existen) y prueba la aplicación:

        ```bash
        npm install
        npm start
        ```

        Deberías ver: `Servidor ejecutándose en http://127.0.0.1:3000/`

    6. Abre otra terminal (sin cerrar la anterior) y prueba con curl:

        ```bash
        curl http://localhost:3000
        ```

    7. Detén la aplicación con `Ctrl+C` y configura NGINX como proxy inverso hacia Node.js:

        ```bash
        nano /etc/nginx/http.d/default.conf
        ```

        ```nginx
        server {
            listen 80;
            server_name _;

            location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        ```

    8. Inicia Node.js en segundo plano y reinicia NGINX:

        ```bash
        npm start > /tmp/nodejs.log 2>&1 &
        /etc/init.d/nginx restart
        ```

    9. Prueba si el servidor responde:

        ```bash
        curl http://localhost
        ```

    **Repite los pasos 1-9 con el otro servidor (webserver2), recordando usar la IP correspondiente (192.168.10.3) en el index.php o index.js.**

### Paso 6: Configuración del Balanceador de Carga en el Proxy (Ubuntu)

Después de configurar ambos servidores web backend, procede a configurar el servidor proxy Ubuntu como balanceador de carga y proxy inverso.

**6.1: Instala NGINX en el servidor Ubuntu (Proxy):**

```bash
sudo apt update
sudo apt install nginx
```

Verifica que NGINX esté corriendo:

```bash
sudo systemctl status nginx
```

**6.2: Crea la configuración del balanceador de carga:**

En Ubuntu 24.04, modifica el archivo configuración del virtual host por defecto en `/etc/nginx/conf.d/`:

```bash
sudo nano /etc/nginx/sites-available/default
```

Reemplaza todo el contenido del archivo por la siguiente configuración:

```nginx
upstream backend {
    # Por defecto usa Round Robin
    # Descomentar para cambiar algoritmo:
    # least_conn;    # Least Connection
    # ip_hash;        # IP Hash (persistencia de sesión)
    
    server 192.168.10.2:80;
    server 192.168.10.3:80;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    location / {
        proxy_pass http://backend;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**6.3: Verifica la sintaxis de la configuración:**

```bash
sudo nginx -t
```

Deberías ver: `nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`

**6.4: Reinicia NGINX:**

```bash
sudo systemctl restart nginx
```

**6.5: Configurar los diferentes algoritmos de balanceo:**

Puedes cambiar entre los tres algoritmos editando el archivo de configuración y descomentando la línea correspondiente:

- **Round Robin (por defecto):** No necesita directiva extra
- **Least Connection:** Descomenta `least_conn;`
- **IP Hash:** Descomenta `ip_hash;`

Después de cualquier cambio, recuerda reinciar NGINX:

```bash
sudo systemctl restart nginx
```

### Paso 7: Verificación y Comparación de Algoritmos de Balanceo

En esta sección, verificarás el funcionamiento del balanceador de carga y compararás los diferentes algoritmos.

**7.1: Obtener la IP del Proxy desde la máquina anfitriona:**

Para acceder al balanceador desde tu máquina anfitriona (Windows/macOS/Linux), necesitas la IP NAT del proxy:

- En la VM Ubuntu (Proxy), ejecuta:

    ```bash
    ip addr show enp0s3 | grep "inet "
    ```

- Anota la IP (probablemente será algo como `10.0.2.15`). Esta es la IP con la que accederás desde tu navegador.

**7.2: Prueba con Round Robin (por defecto):**

El archivo de configuración ya tiene el Round Robin habilitado por defecto.

- Desde tu máquina anfitriona, abre un navegador y ve a: `http://10.0.2.15/` (reemplaza con la IP real del proxy)

- Recarga la página **varias veces** (presiona F5) y observa los cambios en la respuesta

- **Resultado esperado:** El servidor alternará entre "webserver1" y "webserver2" de forma secuencial:
    - Recarga 1 → webserver1 (192.168.10.2)
    - Recarga 2 → webserver2 (192.168.10.3)
    - Recarga 3 → webserver1 (192.168.10.2)
    - Y así sucesivamente...

**7.3: Prueba con Least Connection:**

1. En el servidor Ubuntu, edita la configuración:

    ```bash
    sudo nano /etc/nginx/conf.d/balanceador.conf
    ```

2. Descomenta la línea `least_conn;`:

    ```nginx
    upstream backend {
        least_conn;
        
        server 192.168.10.2:80;
        server 192.168.10.3:80;
    }
    ```

3. Verifica la sintaxis y reinicia:

    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

4. **Prueba con Apache Bench (ab)** para generar múltiples conexiones simultáneamente:

    - En el servidor Ubuntu, instala Apache Bench:

        ```bash
        sudo apt install apache2-utils
        ```

    - Desde el proxy, ejecuta 100 peticiones con 10 concurrentes:

        ```bash
        ab -n 100 -c 10 http://localhost/
        ```

    - **Resultado esperado:** Las conexiones se distribuirán más equitativamente entre ambos servidores, priorizando al que tenga menos conexiones activas.

    - Alterna entre los dos servidores web, apagando uno a la vez, y verifica que el tráfico se redirige al activo:

        ```bash
        ab -n 50 -c 5 http://localhost/
        ```

**7.4: Prueba con IP Hash (Persistencia de Sesión):**

1. Edita la configuración nuevamente:

    ```bash
    sudo nano /etc/nginx/conf.d/balanceador.conf
    ```

2. Reemplaza `least_conn;` con `ip_hash;`:

    ```nginx
    upstream backend {
        ip_hash;
        
        server 192.168.10.2:80;
        server 192.168.10.3:80;
    }
    ```

3. Verifica y reinicia:

    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

4. Desde tu máquina anfitriona:

    - Abre tu navegador (Firefox, Chrome, etc.) y ve a `http://10.0.2.15/`

    - Recarga la página **varias veces** (F5) → **Siempre se conectará al MISMO servidor** (por ejemplo, siempre webserver1)

    - Ahora abre una **ventana de navegación privada/incógnito** e intenta acceder a la misma URL

    - **Resultado esperado:** Con la ventana privada, la IP es la misma, así que también irá al mismo servidor. Sin embargo, si usas una máquina diferente con diferente IP, iría a un servidor diferente.

    - Esto demuestra la **persistencia de sesión** basada en IP: la dirección IP del cliente siempre se mapea al mismo servidor backend.

**7.5: Tabla Resumen de Resultados:**

| Algoritmo | Distribución | Persistencia | Caso de Uso |
|---|---|---|---|
| **Round Robin** | Secuencial (servidor 1, servidor 2, servidor 1...) | No | Servidores con capacidad similar |
| **Least Connection** | Al servidor con menos conexiones activas | No | Servidores con sesiones largas |
| **IP Hash** | Basada en la IP del cliente | Sí | Mantener sesiones del usuario en mismo servidor |

## ⚙️ Sección 3: Práctica en Grupo

En grupos de 3 personas (**5 VMs separados en 3 PCs**: 1 proxy, 2 servidores Web con PHP y 2 servidores Web con NodeJS) o 4 personas (**7 VMs separados en 4 PCs**: 1 proxy, 2 servidores Web con PHP, 2 servidores Web con NodeJS y 2 servidores Web con Python u otro). Deberán configurar una arquitectura con **un balanceador de carga** (NGINX en Ubuntu) y **servidores Web Backend** (NGINX en Alpine), cada uno corriendo una aplicación "Hola Mundo" en PHP o Node.js.

- El balanceador debe usar la política de **least connection**.

- Los servidores web deben estar en la misma red interna.

- El grupo deberá diseñar su propia distribución de direcciones IP dentro de un espacio de red de su elección (ej. `172.16.29.0/27`).

- **El objetivo es que demuestren el funcionamiento del balanceador** de carga de la siguiente manera:

    1. Muestren que las peticiones se distribuyen a los 4 o 6 servidores Web.

    2. Simulen la caída de 2 o 3 de los servidores para demostrar que el tráfico se redirige a los que siguen activos.

    3. Al final, presenten su esquema de red, la configuración de sus servidores y los resultados de sus pruebas de balanceo de carga.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos, que demuestran el dominio de los conceptos y la correcta ejecución de los pasos.

1. **Configuración del Entorno y Preparación (10 pts)**

    - **Configuración de la Red Interna:**

        - Verifica que la red interna `/29` esté correctamente configurada en las cinco/siete máquinas virtuales (Proxy, Servidor 1, Servidor 2, etc).

        - Confirma que las IPs estáticas y la máscara de subred sean correctas (192.168.10.0/29):
            - Proxy: 192.168.10.1
            - WebServer1: 192.168.10.2
            - WebServer2: 192.168.10.3
            - ...

    - **Acceso a Internet desde los Backends:**

        - Demuestra que los servidores backend pueden acceder a internet a través del proxy ejecutando: `ping google.com`

        - Verifica que los servidores pueden descargar paquetes: `apk update`

    - **Instalación y Configuración Base:**

        - NGINX instalado y funcionando en todas las máquinas (proxy + 4/6 servidores web)

        - Servidores web respondiendo con página "Hola Mundo" que muestra nombre del servidor e IP

2. **Práctica Guiada (20 pts)**

    - **Instalación y Configuración del Proxy:**

        - NGINX instalado en Ubuntu
        - Archivo de configuración `/etc/nginx/sites-available/default.conf` presente y bien estructurado
        - Bloque `upstream` correctamente definido con los dos servidores backend

    - **Demostración de Algoritmos (5 puntos cada uno):**

        - **Round Robin:** 
            - Captura de pantalla mostrando alternancia secuencial en navegador
            - Mínimo 5 recargas que demuestren la distribución (web1, web2, web3, web4, web1, web2, etc.)

        - **Least Connection:**
            - Apache Bench instalado y funcional
            - Comando `ab` ejecutado con parámetros de carga: `ab -n 100 -c 10 http://localhost/`
            - Captura mostrando distribución equitativa de conexiones
            - Simulación apagando un servidor para ver redirección de tráfico

        - **IP Hash:**
            - Captura mostrando persistencia: mismo servidor en múltiples recargas
            - Captura de navegación privada/incógnito verificando el comportamiento

3. **Práctica en Grupo y Demostración (40 pts)**

    - **Diseño de la Arquitectura (10 pts):**

        - Diagrama de red documentado (mínimo ASCII, preferentemente visual)
        - Tabla de direcciones IP asignadas a los 5/7 servidores
        - Justificación del espacio de red elegido

    - **Configuración y Operación (15 pts):**

        - 5/7 máquinas virtuales correctamente configuradas y documentadas
        - Servidores web (Alpine) funcionando con "Hola Mundo" (PHP o Node.js)
        - Balanceador de carga (Ubuntu) con algoritmo **least connection** habilitado
        - Archivo de configuración NGINX del proxy

    - **Simulación de Falla y Resiliencia (10 pts):**

        - Demostración práctica: apagar 2-3 servidores manualmente
        - Captura/video mostrando que el balanceador sigue distribuyendo tráfico
        - Verificación de que clientes se redirigen automáticamente a servidores activos
        - Documentación del comportamiento del sistema bajo falla

    - **Presentación y Colaboración (5 pts):**

        - Capacidad para explicar conceptos de proxy inverso y balanceo de carga
        - Descripción detallada de problemas encontrados y soluciones implementadas
        - Evidencia de trabajo en equipo (commits/contribuciones documentadas)

4. **Informe de Laboratorio (30 pts)**

    El informe debe estar en formato markdown (extensión .md) y debe incluir:

    - **Integrantes del Grupo:** Apellidos y nombres de todos los integrantes.
    - **Introducción:** Concepto de proxy inverso y balanceo de carga
    - **Metodología:** Pasos seguidos para la instalación y configuración
    - **Capturas de Pantalla Detalladas:**
        - Tablas de configuración de red (IP, máscara, gateway)
        - Instalación de paquetes en todas las máquinas
        - Configuración de NGINX en cada servidor
        - Pruebas de conectividad (ping entre máquinas)
        - Pruebas de balanceo (Round Robin, Least Connection, IP Hash)
        - Simulación de fallos
    - **Análisis de Resultados:**
        - Comparación entre algoritmos de balanceo
        - Ventajas y desventajas de cada algoritmo
        - Rendimiento del sistema en cada escenario
    - **Conclusiones:** Aprendizajes clave y aplicaciones en producción (de manera individual)
    - **Anexos:** Archivos de configuración completos (NGINX, interfaces de red, etc.)

    **Nota:** Cada uno de los integrantes del grupo deben enviar el informe al eCampus USFX compreso en un archivo ZIP.