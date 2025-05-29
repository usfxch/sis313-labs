# Laboratorio 4.3: Instalación y Configuración de Nginx con PHP-FPM en Ubuntu Server 24.04 LTS

**Objetivo:** Aprender a instalar, configurar, asegurar, optimizar y configurar Server Blocks (equivalente a Virtual Hosts) para el servidor web Nginx en Ubuntu Server 24.04 LTS, incluyendo la integración con PHP-FPM.

**Equipamiento Necesario:**

* Una máquina virtual o física con Ubuntu Server 24.04 LTS instalado.
* Acceso a la línea de comandos con privilegios de superusuario (sudo).
* Un navegador web en tu máquina local para verificar el servidor.

**Pasos:**

**1. Instalación de Nginx:**

   Ejecuta el comando para instalar el servidor web Nginx:

   ```bash
   sudo apt update && sudo apt install nginx
   ```

**2. Verificación del Estado del Servicio Nginx:**

   Verifica que el servicio Nginx esté en ejecución usando el comando:
   
   ```bash
   sudo systemctl status nginx
   ```

   Asegúrate de que el estado sea `active (running)`. Si no lo está, puedes iniciarlo con:

   ```bash
   sudo systemctl start nginx
   ```
   
   Y habilitarlo para el inicio automático con:
   ```bash
   sudo systemctl enable nginx
   ```

**3. Configuración Básica del Firewall (UFW):**

   Si UFW está habilitado, permite el tráfico HTTP (puerto 80) y HTTPS (puerto 443) para Nginx con los comandos:
   ```bash
   sudo ufw allow 'Nginx HTTP'
   ```
   Y para habilitar HTTPS:
   ```bash
   sudo ufw allow 'Nginx HTTPS'
   ```
   O para habilitar ambos (HTTP y HTTPS) utiliza:
   ```bash
   sudo ufw allow 'Nginx Full'
   ```
   Verifica el estado del firewall con:
   ```bash
   sudo ufw status
   ```

**4. Acceso a la Página Web Predeterminada de Nginx:**

   Abre tu navegador web e ingresa la dirección IP de tu servidor Ubuntu. Deberías ver la página de bienvenida predeterminada de Nginx ("Welcome to nginx!").

**5: Exploración de la Configuración de Nginx**

Entender la estructura de los archivos de configuración de Nginx es fundamental para realizar modificaciones y gestionar el servidor web.

* **`/etc/nginx/nginx.conf` (Archivo de Configuración Principal):**
    * **Propósito:** Este es el archivo de configuración global de Nginx. Define la configuración base del servidor, incluyendo los usuarios, el número de procesos de worker, los límites de conexión y la inclusión de otros archivos de configuración.
    * **Contenido Típico:**
        * **Bloque `user`:** Define el usuario y el grupo bajo los cuales se ejecutan los procesos de worker de Nginx.
        * **Bloque `worker_processes`:** Especifica el número de procesos worker que Nginx utilizará. `auto` permite que Nginx determine el número óptimo basado en los recursos del sistema.
        * **Bloque `events`:** Contiene la configuración de manejo de eventos, como el método de conexión (`use epoll;` en Linux) y el número máximo de conexiones por proceso worker (`worker_connections`).
        * **Bloque `http`:** Encierra la configuración del servidor HTTP, incluyendo los tipos MIME, los tiempos de espera, la configuración de los logs y la inclusión de los archivos de configuración de los Server Blocks (`include /etc/nginx/sites-enabled/*;`).
        * **Directivas Generales:** `pid` (ubicación del archivo PID), `include mime.types;` (inclusión de los tipos MIME).
    * **Acción:** Abre este archivo con un editor de texto (`sudo nano /etc/nginx/nginx.conf`) y examina su estructura. Identifica los bloques principales y las directivas mencionadas. Observa cómo se incluyen otros archivos de configuración.

* **`/etc/nginx/sites-available/` (Sitios Disponibles):**
    * **Propósito:** Este directorio contiene los archivos de configuración para cada sitio web (Server Block) que puedes alojar en tu servidor. Cada archivo representa la configuración de un sitio web específico.
    * **Archivo Predeterminado (`default`):** Este es el archivo de configuración del sitio web predeterminado que Nginx sirve si ninguna otra configuración de Server Block coincide con la petición del cliente.
    * **Contenido Típico de un Archivo de Server Block:**
        * **Bloque `server`:** Define la configuración para un sitio web específico. Puede haber múltiples bloques `server` en un mismo archivo o en archivos separados.
        * **`listen`:** Especifica la dirección IP y el puerto en los que este Server Block escuchará las peticiones (por ejemplo, `listen 80;` para HTTP en el puerto 80). También puede especificar `listen [::]:80 ipv6only=on;` para IPv6.
        * **`server_name`:** Define los nombres de dominio para los que este Server Block responderá (por ejemplo, `server_name misitio.local www.misitio.local;`).
        * **`root`:** Especifica el directorio raíz donde se encuentran los archivos del sitio web.
        * **`index`:** Define los archivos que Nginx intentará servir por defecto si se solicita un directorio (por ejemplo, `index index.html index.htm index.php;`).
        * **Bloques `location`:** Definen cómo Nginx maneja las peticiones a diferentes URIs dentro del sitio web. Pueden coincidir con prefijos (`/`), expresiones regulares (`~ \.php$`), etc.
            * **`try_files`:** Intenta servir los archivos en el orden especificado y puede devolver un error 404 si ninguno se encuentra.
            * **`proxy_pass`:** (Para proxy inverso) Especifica la dirección del servidor al que se pasarán las peticiones.
            * **`fastcgi_pass`:** (Para PHP-FPM) Especifica la dirección del socket o la dirección IP y el puerto del proceso PHP-FPM.
            * **`include`:** Incluye otros archivos de configuración (por ejemplo, `include snippets/fastcgi-php.conf;`).
            * **`deny all;` / `allow`:** Controlan el acceso a ciertas ubicaciones.
        * **`access_log` y `error_log`:** Especifican las rutas a los archivos de log para este sitio web.
    * **Acción:** Abre el archivo `default` (`sudo nano /etc/nginx/sites-available/default`) y examina las directivas dentro del bloque `server`. Intenta identificar las directivas mencionadas y comprender cómo se configuran las diferentes ubicaciones.

* **`/etc/nginx/sites-enabled/` (Sitios Habilitados):**
    * **Propósito:** Este directorio contiene enlaces simbólicos a los archivos de configuración de los Server Blocks que están actualmente activos y sirviendo tráfico.
    * **Gestión:** Para habilitar un sitio web configurado en `sites-available/`, se crea un enlace simbólico a su archivo dentro de `sites-enabled/` usando `sudo ln -s /etc/nginx/sites-available/nombredelsitio /etc/nginx/sites-enabled/nombredelsitio`. Para deshabilitar un sitio, se elimina el enlace simbólico usando `sudo rm /etc/nginx/sites-enabled/nombredelsitio`.
    * **Acción:** Lista el contenido de este directorio (`ls -l /etc/nginx/sites-enabled/`) para ver qué sitios están actualmente habilitados.

**6: Configuración de Seguridad y Optimización de Nginx**

Este paso se centra en mejorar la seguridad y el rendimiento de tu servidor Nginx.

* **Ajustar directivas de seguridad en `nginx.conf`:**
    * **`server_tokens off;`:**
        * **¿Por qué?:** Oculta la versión de Nginx en las cabeceras HTTP de respuesta, dificultando que los atacantes conozcan las vulnerabilidades específicas de tu versión.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque `http` y asegúrate de tener la directiva `server_tokens off;`.
    * **`worker_processes auto;`:**
        * **¿Por qué?:** Permite que Nginx determine automáticamente el número óptimo de procesos worker basados en la cantidad de núcleos de CPU disponibles.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque principal (fuera de `http`) y asegúrate de tener `worker_processes auto;`. También puedes especificar un número fijo si lo prefieres.
    * **`worker_connections 1024;`:**
        * **¿Por qué?:** Define el número máximo de conexiones simultáneas que cada proceso worker puede manejar. Ajusta este valor basado en los recursos de tu servidor y el tráfico esperado.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque `events` y ajusta el valor de `worker_connections`.
    * **`sendfile on;`:**
        * **¿Por qué?:** Habilita el uso de la función `sendfile()` del kernel para transferir datos directamente desde el disco a la red, lo que es más eficiente que la copia en espacio de usuario.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque `http` y asegúrate de tener `sendfile on;`.
    * **`tcp_nopush on;`:**
        * **¿Por qué?:** Indica al kernel que no envíe paquetes TCP parcialmente llenos, lo que puede mejorar la eficiencia de la red.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque `http` y asegúrate de tener `tcp_nopush on;`.
    * **`tcp_nodelay on;`:**
        * **¿Por qué?:** Deshabilita el algoritmo de Nagle, que retrasa el envío de paquetes pequeños. Esto puede ser beneficioso para aplicaciones en tiempo real o con muchas peticiones pequeñas.
        * **¿Cómo?:** Edita `/etc/nginx/nginx.conf` dentro del bloque `http` y considera usar `tcp_nodelay on;` si tu aplicación lo requiere.

* **Configurar cabeceras de seguridad en Server Blocks:**
    * **¿Por qué?:** Al igual que con Apache, las cabeceras HTTP pueden mejorar la seguridad del lado del cliente.
    * **¿Cómo?:** Edita el archivo de configuración de tu Server Block (por ejemplo, `/etc/nginx/sites-available/misitio.local`) y dentro del bloque `server`, utiliza la directiva `add_header`.
        * **`add_header X-Frame-Options "SAMEORIGIN";`:** Protege contra Clickjacking. Otras opciones son `DENY` y `ALLOW-FROM uri`.
        * **`add_header X-Content-Type-Options "nosniff";`:** Evita la interpretación incorrecta de tipos MIME.
        * **`add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";`:** (Solo para HTTPS) Fuerza el uso de HTTPS para futuras conexiones.
        * **`add_header Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'self'"`:** Define las fuentes válidas de contenido que el navegador puede cargar para tu sitio. Esta es una directiva compleja y debe configurarse cuidadosamente según las necesidades de tu aplicación.
    * **Importante:** Después de realizar cualquier cambio en los archivos de configuración de Nginx, debes reiniciar el servicio para aplicar los cambios: `sudo systemctl restart nginx`. También puedes verificar la sintaxis de la configuración antes de reiniciar con `sudo nginx -t`.

**7. Configuración de Nginx como Proxy Inverso con PHP-FPM:**

   * **Instalar PHP-FPM y los módulos PHP necesarios:** Utiliza el comando:
        ```bash
        sudo apt install php<tu_version_de_php>-fpm php<tu_version_de_php>-<extension>
        ```
        > Reemplazando las variables con tu versión de PHP y las extensiones requeridas.
   * **Configurar Nginx para pasar las peticiones PHP a PHP-FPM:** Edita el archivo de configuración de tu Server Block y dentro del bloque `location ~ \.php$`, incluye la configuración de `fastcgi_pass` apuntando al socket de PHP-FPM (ejemplo: `fastcgi_pass unix:/run/php/php<tu_version_de_php>-fpm.sock;`) y la inclusión de `snippets/fastcgi-php.conf`.
   * **Reiniciar PHP-FPM y Nginx:** Aplica los cambios con:
     ```bash
     sudo systemctl restart php<tu_version_de_php>-fpm
     ```
     Y para reiniciar:
     ```bash
     sudo systemctl restart nginx
     ```
   * **Probar la configuración de PHP:** Crea un archivo `info.php` en tu `root` (`/var/www/html/`) con `<?php phpinfo(); ?>` y accédelo desde tu navegador. **Elimina este archivo después de la prueba.**

**8. Configuración de Server Blocks en Nginx:**

   * **Crear el directorio para tu sitio web:** Crea un directorio dentro de `/var/www/` para los archivos de tu sitio (ejemplo: `/var/www/misitio.local/public_html`) y ajusta la propiedad y permisos.
   * **Crear el archivo de configuración del Server Block:** Copia el archivo de configuración predeterminado (`default`) y edítalo (ejemplo: `/etc/nginx/sites-available/misitio.local`) para configurar `listen`, `server_name`, `root`, `index`, y los bloques `location` para el manejo de peticiones (incluyendo PHP).
   * **Habilitar el Server Block y deshabilitar el sitio predeterminado:** Crea un enlace simbólico del archivo de configuración en `sites-available/` a `sites-enabled/` usando `sudo ln -s /etc/nginx/sites-available/misitio.local /etc/nginx/sites-enabled/` y elimina el enlace simbólico del sitio predeterminado (`sudo rm /etc/nginx/sites-enabled/default`).
   * **Reiniciar Nginx:** Aplica los cambios con `sudo systemctl restart nginx`.
   * **Configurar tu máquina local para resolver el nombre de dominio (solo para pruebas locales):** Edita el archivo `hosts` de tu sistema operativo y añade una línea que mapee la IP de tu servidor al `server_name` de tu Server Block.

**Práctica Individual (Nginx):**

1.  Crea un Server Block llamado `minginx.local` que sirva archivos desde un directorio diferente dentro de `/var/www/`. Crea un `index.html` de prueba en ese directorio y asegúrate de poder acceder a él a través de tu navegador usando el nombre de dominio `minginx.local` (modificando tu archivo `hosts` local).
2.  Implementa al menos tres de las directivas de seguridad mencionadas en el Paso 6 en la configuración de tu Server Block.
3.  Crea una página PHP sencilla dentro del `root` de tu Server Block que muestre información del servidor (por ejemplo, usando `$_SERVER` variables). Accede a esta página a través de tu navegador.
4.  Documenta los archivos de configuración que creaste y modificaste, explicando las directivas clave y las configuraciones de seguridad implementadas.

**Entrega del Laboratorio (Nginx):**

Para completar este laboratorio, deberás:

* Mostrar los comandos utilizados para instalar Nginx y PHP-FPM.
* Mostrar el estado del servicio Nginx y PHP-FPM después de la instalación y configuración.
* Describir brevemente la función de los archivos de configuración explorados.
* Mostrar la configuración relevante de Nginx para pasar las peticiones a PHP-FPM.
* Incluir el código de la página PHP de prueba.
* Mostrar las directivas de seguridad implementadas en la configuración de Nginx y tu Server Block, con una breve explicación de cada una.
* Mostrar el archivo de configuración de Server Block que creaste, explicando las directivas clave.
* Incluir capturas de pantalla que demuestren el acceso exitoso a tu sitio web de prueba utilizando el nombre de dominio configurado en tu archivo `hosts` local.