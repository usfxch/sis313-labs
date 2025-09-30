# Laboratorio 3.1: Proxy Inverso con Balanceador de Carga Avanzado y Servidores Web NGINX

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Comprender y configurar un proxy inverso y balanceador de carga** con **NGINX**.

- **Diseñar y configurar una red interna** con una máscara de subred `/29` y compartir acceso a internet desde el proxy.

- **Configurar NGINX como proxy inverso** en los servidores backend para servir aplicaciones en PHP o Node.js.

- **Implementar y comparar** los algoritmos de balanceo de carga **round robin**, **least connection** e **IP hash**.

- **Analizar y demostrar el funcionamiento** del balanceo de carga en escenarios reales, incluyendo la simulación de la caída de servidores.


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

                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto $scheme;
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

    10. Después de configurar ambos servidores (webserver1 y webserver2) debes configurar el servidor proxy como Proxy Inverso:

        Edita el archivo principal:

        ```bash
        sudo nano /etc/nginx/sites-enabled/default
        ```

        Borra todo el contenido y añade el siguiente:

        ```nginx
        upstream balanceador {
            server 192.168.10.2:80;
            server 192.168.10.3:80;
        }

        server {
            listen 80 default_server;
            listen [::]:80 default_server;

            server_name _;

            location / {
                proxy_pass http://balanceador;

                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        ```

        Reinicia el servicio de `nginx`:
        ```bash
        sudo systemctl restart nginx
        ```

- **OPCIÓN 2: Instala `nginx` y configura `nginx` como Proxy Inverso:**

    - **Instala Node.js y configura NGINX para que sirva el `index.js`:** En cada servidor Alpine, configura NGINX para que reenvíe las peticiones a un proceso local de Node.js. Crea un archivo `index.js` que muestre un mensaje de "Hola Mundo" e indique el nombre del servidor e IP (ej. "¡Hola Mundo desde el webserver1 (192.168.10.2)!").

### Paso 6: Configuración del balanceador de carga en el proxy (Ubuntu)

- En el servidor Ubuntu, edita el archivo de configuración de NGINX.

- **Define el bloque** `upstream` **con los servidores backend.**

    ```nginx
    upstream backend {
        # ip_hash; o least_conn;
        server 192.168.10.2;
        server 192.168.10.3;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://backend;
        }
    }
    ```

- **Configura los algoritmos de balanceo de carga.**

    - **Round Robin (por defecto):** No necesitas ninguna directiva extra.

    - **Least Connection:** Agrega `least_conn;` en el bloque `upstream`.

    - **IP Hash:** Agrega `ip_hash;` en el bloque `upstream`.

- **Configura el bloque** `server` **principal:**

    - Crea un bloque `server` para el proxy que escuche en el puerto 80.

    - Usa la directiva `proxy_pass` para reenviar las solicitudes al bloque `upstream`.

### Paso 7: Verificación y comparación

- Desde tu máquina anfitriona, accede al balanceador de carga (`<IP de la interfaz NAT del Ubuntu>`).

- **Round Robin:** Realiza varias peticiones consecutivas y observa cómo el tráfico se distribuye de manera secuencial.

- **Least Connection:** Realiza una prueba de carga simulada con una herramienta como `ab` (Apache Bench) para generar muchas conexiones y ver cómo se distribuyen de manera más equitativa.

- **IP Hash:** Cierra y vuelve a abrir tu navegador. Confirma que siempre te conectas al mismo servidor web.

## ⚙️ Sección 3: Práctica en Grupo

En grupos de 3 o 4, deberán configurar una arquitectura con **un balanceador de carga** (NGINX en Ubuntu) y **cinco servidores web** (NGINX en Alpine), cada uno corriendo una aplicación "Hola Mundo" en PHP o Node.js.

- El balanceador debe usar la política de **least connection**.

- Los servidores web deben estar en la misma red interna.

- El grupo deberá diseñar su propia distribución de direcciones IP dentro de un espacio de red de su elección (ej. `172.16.29.0/27`).

- **El objetivo es que demuestren el funcionamiento del balanceador** de carga de la siguiente manera:

    1. Muestren que las peticiones se distribuyen a los 5 servidores.

    2. Simulen la caída de 2 o 3 de los servidores para demostrar que el tráfico se redirige a los que siguen activos.

    3. Al final, presenten su esquema de red, la configuración de sus servidores y los resultados de sus pruebas de balanceo de carga.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos, que demuestran el dominio de los conceptos y la correcta ejecución de los pasos.

1. **Configuración del Entorno y Preparación (10 pts)**

    - **Configuración de la Red Interna:**

        - Verifica que la red interna `/29` esté correctamente configurada en las tres máquinas virtuales (Proxy, Servidor 1 y Servidor 2).

        - Confirma que las IPs estáticas y la máscara de subred sean correctas para cada servidor.

    - **Acceso a Internet desde los Backends:**

        - Demuestra que los servidores backend pueden acceder a internet a través del proxy. Esto se puede verificar con un `ping` a un dominio externo (ej. `google.com`).

    - **Instalación y Configuración Base:**

        - Verifica la correcta instalación de NGINX en todas las máquinas.

        - Confirma que los servidores web NGINX estén configurados como proxy inverso para las aplicaciones "Hola Mundo" en PHP o Node.js.

2. **Práctica Guiada (20 pts)**

    - **Configuración de Balanceo de Carga:**

        - Presenta el archivo de configuración de NGINX en el proxy, con el bloque `upstream` correctamente definido.

    - **Demostración de Algoritmos:**

        - **Round Robin:** Muestra con varias peticiones que el tráfico se distribuye de forma secuencial entre los dos servidores.

        - **Least Connection:** Realiza una prueba de carga simulada para demostrar que NGINX dirige las nuevas conexiones al servidor con menos conexiones activas.

        - **IP Hash:** Demuestra que al refrescar el navegador, la conexión se mantiene en el mismo servidor, garantizando la persistencia de la sesión.

3. **Práctica en Grupo y Demostración (40 pts)**

    - **Diseño de la Arquitectura:**

        - El grupo debe presentar y justificar el diseño de su esquema de red con 5 servidores.

    - **Configuración y Operación:**

        - Demuestra que las 5 máquinas virtuales están correctamente configuradas y que los servidores web están activos.

        - Verifica que el balanceador de carga está configurado con el algoritmo **least connection** para los 5 servidores.

    - **Simulación de Falla:**

        - El grupo debe mostrar, de forma práctica, que el balanceador redirige el tráfico correctamente al apagar 2 o 3 servidores de forma manual. Se evaluará la resiliencia del sistema configurado.

    - **Presentación y Colaboración:**

        - Se evaluará la capacidad del grupo para explicar los pasos, los desafíos encontrados y las soluciones implementadas. La colaboración y el trabajo en equipo son fundamentales.

4. **Informe de Laboratorio (30 pts)**

    El informe debe ser detallado con capturas de pantalla que demuestren cada uno de los pasos realizados.