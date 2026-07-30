# Laboratorio 1.1: Mi Primer Servidor GNU/Linux

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2026

## Introducción para el Estudiante

En Linux, no usamos ventanas ni ratón para administrar servidores. Usamos la **Terminal**. Cada comando es una instrucción precisa. La terminal distingue entre mayúsculas y minúsculas (`Archivo.txt` no es lo mismo que `archivo.txt`).

## 🎯 Objetivos del Laboratorio

- **Administrar servidores GNU/Linux** de manera eficiente a través de la interfaz de línea de comandos (CLI).

- **Dominar la navegación y gestión** del sistema de archivos siguiendo el estándar de jerarquía (FHS).

- **Implementar políticas de seguridad** mediante la administración técnica de usuarios, grupos y permisos de acceso.

- **Gestionar el ciclo de vida del software** y el control de servicios esenciales del sistema (APT y Systemd).

- **Configurar y diagnosticar la conectividad de red** y el acceso remoto seguro mediante SSH y reenvío de puertos.

- **Monitorear recursos y auditar el sistema** utilizando herramientas de rendimiento y análisis de registros (logs) en tiempo real.

## 🛠️ Sección 1: Preparación del Entorno Virtual

En esta etapa, simularemos la **provisión de hardware**. En un entorno real, estarías instalando un servidor físico en un rack; aquí, utilizaremos **VirtualBox** para crear un "servidor virtual" con las especificaciones necesarias para nuestra infraestructura.

**1.1. Creación de la Máquina Virtual (VM)**

Abre VirtualBox y haz clic en el botón **Nueva**. Sigue estas configuraciones cuidadosamente:

- **Nombre y Sistema Operativo:**

    - **Nombre:** `SIS313-Lab1.1`

    - **Imagen ISO:** Busca y selecciona el archivo `.iso` de **Ubuntu Server 24.04.4 LTS** que [descargaste](https://ubuntu.com/download/server).
    
    - **Tipo y Versión:** Verifica que se auto-seleccionen como **Linux** y **Ubuntu (64-bit)**.

    - **Importante:** Marca la casilla **Omitir instalación desatendida**. Esto es vital para que tú mismo realices el proceso de instalación y aprendas cada paso.

    - **Hardware (Recursos del sistema):**

        Ajusta los recursos según la potencia de tu computadora física:

        - **Memoria Base (RAM):**

            - Si tu PC tiene 4 GB de RAM: Asigna **1024 MB**.

            - Si tu PC tiene 6 GB o más: Asigna **2048 MB**.

        - **Procesadores:**

            - Si tu PC tiene hasta 4 núcleos: Asigna **1 CPU**.

            - Si tu PC tiene más de 4 núcleos: Asigna **2 CPUs**.

        - **Disco Duro Virtual:**

            - Selecciona "Crear un disco duro virtual ahora".

            - Tamaño del disco: **20.00 GB**.

**1.2. Instalación del Sistema Operativo**

Inicia la VM haciendo clic en **Flecha Verde (Iniciar)**. El instalador de Ubuntu es puramente textual; usa las flechas del teclado para moverte y la tecla **Enter** para seleccionar.

1. **Idioma:** Selecciona **Spanish** (o Español).

2. **Teclado:** Elige la distribución que coincida con tu teclado físico (usualmente *Spanish* o *Spanish - Latin American*). Puedes probarlo en la sección "Identify keyboard".

3. **Tipo de instalación:** Selecciona la opción por defecto: **Ubuntu Server**.

4. **Red:** El instalador detectará una IP automáticamente (DHCP). No realices cambios, presiona **Hecho**.

5. **Proxy y Mirror:** Deja ambos campos en blanco (por defecto) y presiona **Hecho**.

6. **Configuración de Almacenamiento:**

    - Selecciona **Use an entire disk** (Usar todo el disco).

    - **¡Cuidado!** Asegúrate de que el disco seleccionado sea el de **20 GB**.

    - Presiona **Hecho** y confirma la escritura en el disco cuando se te solicite.

7. **Configuración del Perfil:** Introduce tu nombre, el nombre del servidor (ej. `srv-lab1`), tu nombre de usuario y una contraseña que no olvides (sugerencia: tu documento de identidad).

8. **Upgrade a Ubuntu Pro:** Selecciona **Skip for now** (Omitir por ahora).

9. **Configuración de SSH:** Marca con la barra espaciadora la opción **Install OpenSSH server**. Esto nos permitirá administrar el servidor remotamente más adelante.

10. **Software adicional:** No selecciones nada de la lista. Presiona **Hecho**.

11. **Finalización:** El sistema comenzará a instalarse. Cuando termine, aparecerá la opción **Reboot Now**. Presiona Enter, retira el medio de instalación si VirtualBox te lo pide, y espera a que aparezca el prompt de login.

**1.3. Acceso Remoto: Configuración de Reenvío de Puertos (Port Forwarding)**

Como tu servidor está dentro de una red interna de VirtualBox (NAT), tu computadora física no "ve" directamente al servidor. Para administrarlo de forma profesional, vamos a crear un "puente" o túnel que conecte un puerto de tu PC con el puerto de servicio SSH del servidor.

1. **Configuración en VirtualBox**

    1. Con la VM **SIS313-Lab1.1** seleccionada (puede estar encendida o apagada), haz clic en **Configuración**.

    2. Ve al menú **Red**.

    3. Asegúrate de que en el "Adaptador 1" esté seleccionado **Conectado a: NAT**.

    4. Haz clic en el botón desplegable **Avanzadas**.

    5. Haz clic en el botón **Reenvío de puertos**.

    6. En la ventana que aparece, haz clic en el icono de **Agregar nueva regla** (el símbolo `+` verde a la derecha) y completa los datos exactamente así:

        | Nombre | Protocolo | IP anfitrión | Puerto anfitrión | IP invitado | Puerto invitado |
        | - | - | - | - | - | - |
        | SSH | TCP |   | 2222 | 10.0.2.15 | 22 |

        **¿Qué significa esto?** Le estamos diciendo a VirtualBox: "*Cualquier petición que llegue a mi PC real por el puerto 2222, envíala automáticamente al puerto 22 (SSH) de mi servidor virtual*".

    7. Haz clic en **Aceptar** en ambas ventanas para guardar.

2. **Instalación de la Terminal y Prueba de Conexión**

    Para administrar servidores, los profesionales no suelen usar la ventana pequeña de VirtualBox, sino una terminal moderna.

    1. **Instala Warp:** Descarga e instala [Warp Terminal](https://app.warp.dev/referral/3DY6RJ). Es una terminal inteligente que te ayudará mucho en este semestre.

    2. **Prueba la conexión:** Abre Warp en tu PC física y escribe el siguiente comando (reemplaza `marcelo` por el nombre de usuario que elegiste durante la instalación):

        ``` bash
        ssh -p 2222 marcelo@127.0.0.1
        ```
    3. **Primer acceso:** La terminal te preguntará: *"Are you sure you want to continue connecting (yes/no/[fingerprint])?"*. Escribe **yes** y presiona **Enter**.

    4. **Contraseña:** Introduce la contraseña que configuraste en la instalación. **Nota:** No verás asteriscos ni puntos mientras escribes por seguridad; solo escribe y presiona **Enter**.

    Si lograste entrar, verás que el texto de la terminal cambia (ej. `marcelo@srv-lab1:~$`). ¡Felicidades, ya estás administrando tu servidor de forma remota!

## 🖥️ Sección 2: Práctica Guiada (Administración de Sistemas)

Ahora que estás conectado a tu servidor mediante **Warp** vía SSH, verás una línea de texto que espera tus órdenes (el *prompt*). En un servidor, no hay íconos; todo se hace mediante verbos y sustantivos (comandos y rutas).

**2.1. Gestión de Archivos y Directorios: "Navegando en el Almacenamiento"**

Un administrador debe saber organizar la información. En Linux, todo es un archivo.

- `pwd` **(Print Working Directory)**: Te dice en qué lugar del mundo (del servidor) estás.

    ```bash
    pwd
    ```
    > Debería devolver `/home/tu_usuario`.

- `ls` (List): Lista el contenido de una carpeta.

    ```bash
    ls -la
    ```
    > La `-l` es formato largo y la `-a` muestra archivos ocultos.

- `mkdir` **(Make Directory)**: Crea carpetas para organizar servicios.

    ```bash
    mkdir -p lab1/respaldos
    ```
    > El `-p` crea toda la ruta si no existe.

- `touch`: Crea un archivo vacío. Útil para "marcar" archivos de prueba.

    ```bash
    touch lab1/notas.txt
    ```

- `cp` **(Copy)** y `mv` **(Move/Rename)**:

    ```bash
    cp lab1/notas.txt lab1/notas_backup.txt
    ```

    ```bash
    mv lab1/notas.txt /tmp/
    ```
    > Mueve el archivo a la carpeta temporal.

    ```bash
    mv lab1/notas_backup.txt lab1/final.txt
    ```
    > Renombrar o cambiar el nombre de un archivo.

- `rm` **(Remove)**: Borra archivos. ¡Cuidado! En Linux no hay papelera de reciclaje.

    ```bash
    rm lab1/final.txt
    ```
    > Para borrar carpetas con contenido usa `rm -rf`.

- `head` y `tail`: Para leer el principio o el final de archivos largos (como logs).

    ```bash
    tail -f /var/log/syslog
    ```
    > El `-f` permite ver en tiempo real cómo se escribe el archivo.

- `grep` y `find`: Los buscadores del sistema.

    ```bash
    find /etc -name "hostname"
    ```
    > Busca un archivo llamado hostname en `/etc`.

    ```bash
    grep "error" /var/log/syslog
    ```
    > Filtra todas las líneas que digan "error".

**2.2. Administración de Usuarios y Grupos: "Acceso Lógico"**

En infraestructura, nunca compartimos contraseñas. Cada departamento tiene su grupo y cada empleado su usuario.

- `groupadd` / `groupdel`: Gestión de grupos.

    ```bash
    sudo groupadd ventas
    ```
    > Crea el grupo Ventas.

    ```bash
    sudo groupadd -g 2000 contabilidad
    ```
    > Crea el grupo con un ID específico.

- `useradd` / `usermod`: Creación de usuarios.

    ```bash
    sudo useradd -u 2001 -g contabilidad -md /home/jperez -s /bin/bash -c "Juan Perez" jperez
    ```
    Ejemplo completo donde: 
    - `-u`: Especifica el ID de usuario.

    - `-g`: Le asigna al grupo contabilidad.
    
    - `-md`: Crea y selecciona su carpeta personal.

    - `-s`: Le da una terminal (bash).

    - `-c`: Añade un comentario con su nombre real.

- `passwd` y `chage`: Seguridad de contraseñas.

    ```bash
    sudo passwd jperez
    ```
    > Asigna contraseña.

    ```bash
    sudo chage -M 90 jperez
    ```
    > Obliga a cambiar la contraseña cada 90 días.

**2.3. El Comando `sudo` y Gestión de Paquetes: "Instalando la Capa de Software"**

- `sudo` es el prefijo que te da poderes de "Dios" (Root) temporalmente. Sin él, no puedes modificar el sistema.

- `apt` **(Advanced Package Tool)**: Es tu "tienda de aplicaciones".

    ```bash
    sudo apt update
    ```
    > Actualiza la lista de software disponible. **Hazlo siempre primero**.

    ```bash
    sudo apt upgrade
    ```
    > Descarga e instala las actualizaciones de seguridad del SO.

    ```bash
    sudo apt install nginx
    ```
    > Instala el servidor web Nginx.

    ```bash
    sudo apt remove nginx
    ```
    > Borra el programa pero deja la configuración.

    ```bash
    sudo apt purge nginx
    ```
    > Borra el programa y todos sus archivos de configuración.

**2.4. Control de Servicios: `systemctl`**

Una vez instalado un programa (como `Nginx`), este corre como un "demonio" o servicio en segundo plano.

- `status`: Verifica si el servicio está vivo.

    ```bash
    sudo systemctl status nginx
    ```

- `stop` / `start` / `restart`: detiene, inicia y/o reinicia.

    ```bash
    sudo systemctl status nginx
    ```

    ```bash
    sudo systemctl stop nginx
    ```
    >  Apaga el servidor web    .

- `enable` / `disable`:

    ```bash
    sudo systemctl enable nginx
    ```
    > Hace que el servidor web encienda solo cuando se prenda la computadora.

**2.5. Permisos de Archivos: `chmod` y `chown`**

Esto determina quién puede ver o modificar qué. Es la base de la seguridad en el Data Center.

- `chown` `(Change Owner)`: Cambia el dueño y el grupo.

    ```bash
    sudo chown jperez:contabilidad informe.txt
    ```

- `chmod` **(Change Mode)**: Cambia los permisos (Lectura=4, Escritura=2, Ejecución=1).

    ```bash
    sudo chmod 640 informe.txt
    ```

    Donde:
    - `6` (4+2): Dueño puede leer y escribir.

    - `4`: Grupo puede solo leer.

    - `0`: El resto del mundo no puede hacer nada.

**2.6. Comandos de Red: "Verificando la Conectividad"**

Un servidor desconectado no sirve de nada.

- `ip addr`: Te muestra tu dirección IP real. Busca la que está bajo `eth0` o `enp0s3`.

    ```bash
    ip addr
    ```

- `ping`: Verifica si puedes "ver" a otro equipo.

    ```bash
    ping -c 4 google.com
    ```
    > Envía 4 paquetes a Google.

- `ss` o `netstat`: Muestra qué puertos están abiertos en tu servidor (quién está escuchando).

    ```bash 
    ss -tunlp
    ```
    > Muestra puertos TCP/UDP activos y qué programa los usa.

## 🏗️ Sección 3: Práctica  Individual (Desafío de Infraestructura – "El Nodo de Servicios USFX")

**Contexto:** La Universidad necesita configurar un servidor de pruebas para el departamento de desarrollo. Tu misión es preparar el entorno lógico, instalar los servicios base y garantizar que los accesos sean restrictivos y seguros.

**El Escenario Técnico**

Debes realizar las siguientes tareas de forma autónoma utilizando la terminal (vía Warp/SSH).

**Ejercicio 1: Estructura de Proyecto**

1. Crea una carpeta en la raíz del sistema llamada `/srv/plataforma`.

2. Dentro de esa carpeta, crea la siguiente subestructura de un solo golpe (usando un solo comando):

    - `/srv/plataforma/codigo` (Para los archivos fuente).

    - `/srv/plataforma/config` (Para archivos de configuración).

    - `/srv/plataforma/logs` (Para registros de auditoría).

**Ejercicio 2: Gestión de Identidades y Accesos**

Implementa el siguiente esquema de seguridad para el personal:

| Rol | Grupo | Usuario |
| - | - | - |
| Desarrollador	| `devs` | `dev_user` |
| Administrador | `admins_ti` | `admin_user` |

Para ello, sigue las instrucciones: 

1. Crea los grupos y usuarios correspondientes. Asegúrate de que ambos tengan acceso a la terminal Bash.

2. **Configuración de Permisos Críticos:**

    - La carpeta `/srv/plataforma/codigo` debe pertenecer al usuario `dev_user` y al grupo `devs`. El dueño debe tener control total, el grupo solo lectura y el resto del mundo nada.

    - La carpeta `/srv/plataforma/config` debe ser **privada** solo para el usuario `admin_user`. Nadie más (excepto `root`) debe poder siquiera listar su contenido.

    - **Reto:** Configura la carpeta `/srv/plataforma/logs` de modo que el grupo `devs` pueda entrar a ver los archivos, pero **no pueda crear archivos nuevos ni borrarlos**.

**Ejercicio 3: Provisión de Software y Servicios**

1. Actualiza la lista de repositorios de tu servidor.

2. Instala el servidor web **Apache** y el editor de texto **Vim** (o **Nano** si prefieres).

3. Verifica que Apache esté "activo" y configurado para iniciar siempre que el servidor se encienda.

4. **Simulación de Mantenimiento:** Detén el servicio de Apache, verifica su estado (debe salir `inactive`) y luego vuelve a iniciarlo.

**Ejercicio 4: Diagnóstico de Red y Hostname**

1. Cambia el nombre del servidor (hostname) a `nodo-desarrollo-usfx`.

2. Identifica la dirección IP privada de tu servidor.

3. Usa el comando `ss` o `netstat` para demostrar que el servidor Apache está escuchando correctamente en el puerto 80.

4. Realiza un `ping` (solo 3 paquetes) a `google.com` para verificar que el servidor tiene salida a internet.

**Ejercicio 5: Auditoría, Monitoreo y Visualización de Logs**

1. **Búsqueda:** Encuentra todos los archivos en `/etc` que terminen en `.conf` y guarda la lista en `/home/tu_usuario/reporte_config.txt`.

2. **Monitoreo:** Instala y ejecuta `htop` para visualizar el consumo de recursos en tiempo real.

3. **Visualización de Logs:** Abre una segunda pestaña en tu terminal Warp y conéctate por SSH.

    - Ejecuta el comando `tail -f /var/log/apache2/access.log`.

    - En la primera pestaña, intenta acceder a la IP de tu servidor (o usa el comando `curl localhost`).

    - Observa y captura cómo el log se actualiza en tiempo real en la segunda pestaña al recibir la petición.

### 📄 Instrucciones para los Entregables

Para aprobar este laboratorio, debes presentar un **Informe Técnico en formato PDF** que cumpla estrictamente con los siguientes puntos:

1. **Estructura del Informe:**

    - Portada (Nombre, Carrera, Asignatura).

    - Desarrollo: Una sección por cada una de las **5 ejercicios** planteadas arriba.

2. **Requisitos de las Capturas de Pantalla:**

    - Deben ser claras y legibles (se recomienda el uso de modo oscuro en la terminal para mejor contraste).

    - **Descripción Obligatoria:** Debajo de cada captura, debes escribir una breve explicación del comando utilizado y qué resultado se está observando.

    - *Ejemplo:* "En la imagen superior se observa el comando `ls -l /srv/plataforma`, donde se verifican los permisos asignados a los grupos de desarrollo."

3. **Conclusión Final:**

    - Al finalizar el documento, incluye una breve conclusión personal (mínimo 10 líneas) sobre la importancia de la administración por línea de comandos en la gestión de infraestructura de TI.