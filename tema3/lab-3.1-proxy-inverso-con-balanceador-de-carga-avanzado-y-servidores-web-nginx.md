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

    - **Sistema Operativo:** Ubuntu Server 24.04.

    - **Interfaces de Red:**

        - `enp0s3` (NAT): Permite el acceso a internet. Configura un reenvío de puertos (por ejemplo, del puerto 2222 del host al puerto 22 de la MV) para acceso SSH.

        - `enp0s8` (Red Interna): Se conectará a los servidores web del backend.

            - **Configuración de la red:**

                - IP: `192.168.10.1`

                - Máscara de subred: `255.255.255.248` (equivalente a `/29`).

2. **Máquina Virtual 2: Servidor Web 1**

    - **Sistema Operativo:** Alpine Linux ([descargar aquí](https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-standard-3.22.1-x86_64.iso)).

    - **Interfaces de Red:**

        - `eth0` (Red Interna): Se conectará a la red interna del proxy.

            - **Configuración de la red:**

                - IP: `192.168.10.2`

                - Máscara de subred: `255.255.255.248`

                - Gateway: `192.168.10.1`

3. **Máquina Virtual 3: Servidor Web 2**

    - **Sistema Operativo:** Alpine Linux ([descargar aquí](https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-standard-3.22.1-x86_64.iso)).

    - **Interfaces de Red:**

        - `eth0` (Red Interna): Conectada a la misma red interna que los otros servidores.

            - **Configuración de la red:**

                - IP: `192.168.10.3`

                - Máscara de subred: `255.255.255.248`

                - Gateway: `192.168.10.1`

## 💻 Sección 2: Práctica guiada

### Paso 1: Instalación y Configuración de Red durante la instalación del S.O.

- **En Ubuntu Server (Proxy):** Durante la instalación, configura la red de forma manual. Deja la interfaz NAT (`enp0s3`) en **DHCP** para obtener una IP automáticamente y poder acceder a internet. Para la interfaz de red interna (`enp0s8`), configúrala con la **dirección IP estática** `192.168.10.1` y la máscara `/29`.

- **En Alpine Linux (Servidores Backend):** Durante la instalación, configura la interfaz de red (`eth0`) de cada servidor con la **dirección IP estática** correspondiente (`192.168.10.2` para el Servidor 1 y `192.168.10.3` para el Servidor 2). Asegúrate de configurar la **máscara de subred** `/29` y el **gateway** como la IP del proxy (`192.168.10.1`).

### Paso 2: Compartir Acceso a Internet desde el Proxy

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

### Paso 3: Configuración de los servidores web backend

- **Verifica la conexión a internet:** Desde los servidores Alpine, haz un `ping google.com` para confirmar que tienen acceso a internet.

- **Instala NGINX y el entorno de ejecución:**

    - Para PHP: Instala `nginx` y `php-fpm`.

    - Para Node.js: Instala Node.js y configura NGINX para que sirva el `index.js`.

- **Configura NGINX como proxy inverso local:** En cada servidor Alpine, configura NGINX para que reenvíe las peticiones a un proceso local de PHP-FPM o Node.js. Crea un archivo `index.php` o `index.js` que muestre un mensaje de "Hola Mundo" e indique el nombre del servidor (ej. "¡Hola Mundo desde el Servidor 1!").

### Paso 4: Configuración del balanceador de carga en el proxy (Ubuntu)

- En el servidor Ubuntu, edita el archivo de configuración de NGINX.

- **Define el bloque** `upstream` **con los servidores backend.**

    ```nginx
    upstream backend_servers {
        server 10.0.0.2;
        server 10.0.0.3;
    }
    ```

- **Configura los algoritmos de balanceo de carga.**

    - **Round Robin (por defecto):** No necesitas ninguna directiva extra.

    - **Least Connection:** Agrega `least_conn;` en el bloque `upstream`.

    - **IP Hash:** Agrega `ip_hash;` en el bloque `upstream`.

- **Configura el bloque** `server` **principal:**

    - Crea un bloque `server` para el proxy que escuche en el puerto 80.

    - Usa la directiva `proxy_pass` para reenviar las solicitudes al bloque `upstream`.

### Paso 5: Verificación y comparación

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