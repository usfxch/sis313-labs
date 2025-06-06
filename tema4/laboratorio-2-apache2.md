# Laboratorio 4.2: Instalación y Configuración de Apache2 con PHP en Ubuntu Server LTS

**Objetivo:** Aprender a instalar, configurar, asegurar, optimizar y configurar Virtual Hosts para el servidor web Apache en Ubuntu Server 24.04 LTS, incluyendo la integración con PHP como módulo.

**Equipamiento Necesario:**

* Una máquina virtual o física con Ubuntu Server 24.04 LTS instalado.
* Acceso a la línea de comandos con privilegios de superusuario (sudo).
* Un navegador web en tu máquina local para verificar el servidor.

**Pasos:**

**1. Instalación de Apache:**

   Ejecuta el comando para instalar el servidor web Apache2:

   ```bash
   sudo apt update && sudo apt install apache2
   ```

**2. Verificación del Estado del Servicio Apache:**

   Verifica que el servicio Apache esté en ejecución usando el comando:

   ```bash
   sudo systemctl status apache2
   ```
   Asegúrate de que el estado sea `active (running)`. Si no lo está, puedes iniciarlo con:
   
   ```bash
   sudo systemctl start apache2
   ```
   
   O habilitarlo para el inicio automático con:
   
   ```bash
   sudo systemctl enable apache2
   ```

**3. Configuración Básica del Firewall (UFW):**

   Si UFW está habilitado, permitirá el tráfico HTTP (puerto 80) y HTTPS (puerto 443) para Apache con el comando:
   
   ```bash
   sudo ufw allow 'Apache'
   ```
   
   Verifica el estado del firewall con:
   
   ```bash
   sudo ufw status
   ```

**4. Acceso a la página Web predeterminada de Apache:**

   Abre tu navegador web e ingresa la dirección IP de tu servidor Ubuntu. Deberías ver la página de bienvenida predeterminada de Apache2.

**5: Exploración de la configuración de Apache**

Al explorar los archivos de configuración, obtendrás una base para las modificaciones posteriores.

* **`/etc/apache2/apache2.conf` (Archivo de Configuración Principal):**
    * **Propósito:** Este es el archivo de configuración global de Apache. Define los ajustes del servidor que afectan a todas las operaciones, a menos que se sobrescriban en otros archivos de configuración (como los de los Virtual Hosts).
    * **Contenido Típico:**
        * **Directivas Generales:** `ServerRoot` (la raíz del servidor), `PidFile` (la ubicación del archivo del ID del proceso), `Timeout` (tiempo de espera para las peticiones), `KeepAlive` (habilitar/deshabilitar conexiones persistentes), `MaxKeepAliveRequests`, `KeepAliveTimeout`.
        * **Configuración de Módulos:** Directivas `<IfModule>` para configurar módulos específicos. La carga de los módulos se realiza a través de directivas `Include` que referencian los archivos `.load` en `/etc/apache2/mods-enabled/`.
        * **Configuración de Inclusión:** Directivas `Include` para incorporar otros archivos de configuración, como los de los Virtual Hosts (`/etc/apache2/sites-enabled/*.conf`) y la configuración específica de los puertos (`/etc/apache2/ports.conf`).
        * **Configuración de Directorios Globales:** Bloques `<Directory>` que definen permisos y opciones para los directorios del sistema de archivos a los que Apache tiene acceso. El bloque `<Directory />` define la configuración predeterminada para todos los directorios.
    * **Acción:** Abre este archivo con un editor de texto (`sudo nano /etc/apache2/apache2.conf`) y léelo detenidamente. Intenta identificar las directivas mencionadas anteriormente y observa cómo se estructuran las inclusiones de otros archivos.

* **`/etc/apache2/ports.conf` (Configuración de Puertos):**
    * **Propósito:** Este archivo define los puertos en los que Apache escucha las peticiones entrantes. Por defecto, Apache escucha en el puerto 80 para HTTP y puede configurarse para escuchar en el puerto 443 para HTTPS.
    * **Contenido Típico:** Directivas `Listen 80` y `Listen 443`. También puede contener la configuración de la interfaz de red específica en la que Apache escucha (por ejemplo, `Listen 192.168.1.100:80`).
    * **Importancia:** Si necesitas cambiar el puerto predeterminado de Apache, este es el archivo que debes modificar.
    * **Acción:** Abre este archivo (`sudo nano /etc/apache2/ports.conf`) y verifica en qué puertos está configurado para escuchar Apache.

* **`/etc/apache2/sites-available/` (Sitios Disponibles):**
    * **Propósito:** Este directorio contiene los archivos de configuración para cada sitio web (Virtual Host) que puedes alojar en tu servidor. Cada archivo `.conf` representa la configuración de un sitio web específico.
    * **Archivo Predeterminado (`000-default.conf`):** Este es el archivo de configuración del sitio web predeterminado que Apache sirve si no coincide con ningún otro Virtual Host configurado.
    * **Contenido Típico de un Archivo de Virtual Host:**
        * **`<VirtualHost *:80>` (o `<VirtualHost *:443>` para HTTPS):** Delimita la configuración para un Virtual Host específico que responde en el puerto indicado. El `*` indica que se aplica a todas las interfaces de red del servidor.
        * **`ServerAdmin`**: La dirección de correo electrónico del administrador del servidor.
        * **`ServerName`**: El nombre de dominio principal para este sitio web (por ejemplo, `misitio.local`).
        * **`ServerAlias`**: Nombres de dominio alternativos para este sitio web (por ejemplo, `www.misitio.local`).
        * **`DocumentRoot`**: La ruta al directorio en el sistema de archivos donde se almacenan los archivos del sitio web.
        * **`ErrorLog`**: La ruta al archivo donde se registran los errores del servidor para este sitio web.
        * **`CustomLog`**: La ruta al archivo donde se registran las peticiones de acceso al servidor para este sitio web.
        * **`<Directory /ruta/al/documentroot>`**: Define las directivas de configuración que se aplican al directorio especificado y sus subdirectorios. Esto incluye opciones como `Options Indexes FollowSymLinks`, `AllowOverride All`, `Require all granted` (para permisos de acceso).
    * **Acción:** Abre el archivo `000-default.conf` (`sudo nano /etc/apache2/sites-available/000-default.conf`) y familiarízate con las directivas dentro del bloque `<VirtualHost>`.

* **`/etc/apache2/sites-enabled/` (Sitios Habilitados):**
    * **Propósito:** Este directorio contiene enlaces simbólicos a los archivos de configuración de los Virtual Hosts que están actualmente activos y sirviendo tráfico.
    * **Gestión:** Para habilitar un sitio web configurado en `sites-available/`, se crea un enlace simbólico a su archivo `.conf` dentro de `sites-enabled/` usando el comando `a2ensite`. Para deshabilitar un sitio, se elimina el enlace simbólico usando `a2dissite`.
    * **Acción:** Lista el contenido de este directorio (`ls -l /etc/apache2/sites-enabled/`) para ver qué sitios están actualmente habilitados.

**6: Configuración de Seguridad y Optimización de Apache**

Mejoras a la seguridad y el rendimiento de tu servidor Apache.

* **Deshabilitar módulos innecesarios:**
    * **¿Por qué?:** Habilitar solo los módulos que tu aplicación necesita reduce la superficie de ataque y el consumo de recursos.
    * **¿Cómo?:**
        * Lista los módulos habilitados:

          ```bash
          apachectl -M
          ```
          > Identifica los módulos que no necesitas (investiga si no estás seguro de su función).
        * Deshabilita un módulo:
          ```bash
          sudo a2dismod <nombre_del_modulo>
          ```
          Por ejemplo:
          ```bash
          sudo a2dismod status info
          ```
        * Reinicia Apache:
          ```bash
          sudo systemctl restart apache2
          ```

* **Ajustar directivas de seguridad en `apache2.conf`:**
    * **`ServerTokens Prod`:**
        * **¿Por qué?:** Oculta la versión detallada de Apache en las cabeceras HTTP de respuesta. Esto dificulta que los atacantes conozcan las vulnerabilidades específicas de tu versión.
        * **¿Cómo?:** Edita `/etc/apache2/apache2.conf` y cambia la línea `ServerTokens OS` o `ServerTokens Full` a `ServerTokens Prod`.
    * **`ServerSignature Off`:**
        * **¿Por qué?:** Evita que Apache muestre información del servidor (versión, sistema operativo) en las páginas de error generadas por el servidor.
        * **¿Cómo?:** Edita `/etc/apache2/conf-enabled/security.conf` y cambia la línea `ServerSignature On` a `ServerSignature Off`.
    * **`Timeout`:**
        * **¿Por qué?:** Controla el tiempo que el servidor espera entre la recepción de paquetes para una solicitud. Un valor demasiado alto puede mantener conexiones inactivas abiertas y consumir recursos.
        * **¿Cómo?:** Edita `/etc/apache2/apache2.conf` y ajusta el valor en segundos (por ejemplo, `Timeout 300`).
    * **`KeepAlive Off` / Ajustar `KeepAliveTimeout` y `MaxKeepAliveRequests`:**
        * **`KeepAlive Off`:** Deshabilita las conexiones persistentes. Puede reducir el consumo de recursos para sitios con muchas peticiones cortas.
        * **`KeepAliveTimeout`:** El tiempo (en segundos) que el servidor esperará la siguiente petición en una conexión persistente. Un valor bajo libera recursos más rápido.
        * **`MaxKeepAliveRequests`:** El número máximo de peticiones que se permitirán en una conexión persistente. Un valor bajo puede ayudar a prevenir el consumo excesivo de recursos por una sola conexión.
        * **¿Cómo?:** Edita `/etc/apache2/apache2.conf` y ajusta estas directivas según tu análisis de tráfico.

* **Configurar cabeceras de seguridad en Virtual Hosts:**
    * **¿Por qué?:** Las cabeceras HTTP pueden proporcionar instrucciones de seguridad adicionales a los navegadores de los clientes.
    * **¿Cómo?:** Edita el archivo de configuración de tu Virtual Host (por ejemplo, `/etc/apache2/sites-available/misitio.local.conf`) y dentro del bloque `<VirtualHost>` (o dentro de un bloque `<Directory>` específico si quieres aplicarlo a una parte del sitio), añade las directivas `Header always set ...`.
        * **`X-Frame-Options "SAMEORIGIN"`:** Impide que tu sitio sea incrustado en un `<iframe>` por sitios de otros dominios, protegiendo contra Clickjacking. Otras opciones son `DENY` y `ALLOW-FROM uri`.
        * **`X-Content-Type-Options "nosniff"`:** Indica al navegador que no intente adivinar el tipo MIME de un recurso basándose en su contenido. Esto ayuda a prevenir ataques donde un atacante sube un archivo malicioso con una extensión diferente.
        * **`Strict-Transport-Security "max-age=31536000; includeSubDomains"`:** (Solo para HTTPS) Fuerza a los navegadores a usar HTTPS para futuras conexiones al dominio durante el tiempo especificado (`max-age`). `includeSubDomains` aplica esta política a todos los subdominios.
    * **Importante:** Después de realizar cualquier cambio en los archivos de configuración de Apache, siempre debes reiniciar el servicio para aplicar los cambios:
      ```bash
      sudo systemctl restart apache2
      ```
      También es recomendable verificar 
      la sintaxis de la configuración antes de reiniciar con:
      ```bash
      sudo apachectl configtest
      ```

**7. Configuración de PHP como Módulo (mod_php):**

   * **Instalar `libapache2-mod-php` y los módulos PHP necesarios:** Utiliza el comando:
      ```bash
      sudo apt install libapache2-mod-php<tu_version_de_php> php<tu_version_de_php> php<tu_version_de_php>-<extension> ...
      ```
      > Reemplaza las variables con tu versión de PHP y las extensiones requeridas. Por ejemplo:
       ```bash
      sudo apt install libapache2-mod-php8.3 php8.3 php8.3-mysql
      ```
   * **Habilitar el módulo PHP (si no se habilitó automáticamente):** Usa el comando:
      ```bash
      sudo a2enmod php<tu_version_de_php>
      ```
   * **Configurar Apache para procesar archivos PHP:** Asegúrate de que la configuración de tu virtual host incluya la directiva: 
      ```bash
      <FilesMatch \.php$>
         SetHandler application/x-httpd-php
      </FilesMatch>
      ```
      > Esto debe ir dentro del bloque `<Directory>` de tu `DocumentRoot`.
   * **Reiniciar Apache:** Aplica los cambios con:
      ```bash
      sudo systemctl restart apache2
      ```
   * **Probar la configuración de PHP:** Crea un archivo `info.php` en tu `DocumentRoot` con:
      ```php
      <?php phpinfo(); ?>
      ```
      > Accédelo desde tu navegador.

**8. Configuración de Virtual Hosts en Apache:**

   * **Crear el directorio para tu sitio web:** Crea un directorio dentro de `/var/www/` para los archivos de tu sitio (ejemplo: `/var/www/misitio.local/public_html`) y ajusta la propiedad y permisos.
   * **Crear el archivo de configuración del Virtual Host:** Copia el archivo de configuración predeterminado (`000-default.conf`) y edítalo (ejemplo: `/etc/apache2/sites-available/misitio.local.conf`) para configurar `ServerAdmin`, `ServerName`, `ServerAlias`, `DocumentRoot`, `ErrorLog`, `CustomLog`, y las directivas dentro del bloque `<Directory>`.
   * **Habilitar el Virtual Host y deshabilitar el sitio predeterminado:** Usa los comandos:
      ```bash
      sudo a2ensite misitio.local.conf
      sudo a2dissite 000-default.conf
      ```
   * **Reiniciar Apache:** Aplica los cambios con:
      ```bash
      sudo systemctl restart apache2
      ```
   * **Configurar tu máquina local para resolver el nombre de dominio (solo para pruebas locales):** Edita el archivo `hosts` de tu sistema operativo y añade una línea que mapee la IP de tu servidor al `ServerName` y `ServerAlias` de tu Virtual Host.

**Práctica Individual (Apache):**

1.  Crea un Virtual Host llamado `miapache.local` que sirva archivos desde el directorio `/var/www/html/miapache`. Crea un `index.html` de prueba en ese directorio y asegúrate de poder acceder a él a través de tu navegador usando el nombre de dominio `miapache.local` (modificando tu archivo `hosts` local).
2.  Implementa al menos tres de las directivas de seguridad mencionadas en el Paso 6 en la configuración de tu Virtual Host.
3.  Crea una página PHP sencilla dentro del `DocumentRoot` de tu Virtual Host que muestre información del servidor (por ejemplo, usando `$_SERVER` variables). Accede a esta página a través de tu navegador.
4.  Documenta los archivos de configuración que creaste y modificaste, explicando las directivas clave y las configuraciones de seguridad implementadas.

**Entrega del Laboratorio (Apache):**

Para completar este laboratorio, deberás:

* Mostrar los comandos utilizados para instalar Apache y PHP.
* Mostrar el estado del servicio Apache después de la instalación y configuración.
* Describir brevemente la función de los archivos de configuración explorados.
* Mostrar la configuración relevante de Apache para habilitar PHP.
* Incluir el código de la página PHP de prueba.
* Mostrar las directivas de seguridad implementadas en la configuración de Apache y tu Virtual Host, con una breve explicación de cada una.
* Mostrar el archivo de configuración de Virtual Host que creaste, explicando las directivas clave.
* Incluir capturas de pantalla que demuestren el acceso exitoso a tu sitio web de prueba utilizando el nombre de dominio configurado en tu archivo `hosts` local.