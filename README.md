# Infraestructura, Plataformas Tecnológicas y Redes (SIS313) - Índice de Laboratorios

**Universidad San Francisco Xavier de Chuquisaca**

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

---

### Tema 1: Fundamentos y Diseño de Infraestructura de TI

- [Laboratorio 1.1: Diseño de un centro de datos virtualizado y plan de continuidad operacional](tema1/lab-1.1-diseno-de-un-centro-de-datos.md)

- [Laboratorio 1.2: Repaso a comandos de administración de GNU/Linux](tema1/lab-1.2-administracion-monitoreo-y-rendimiento.md)


### Tema 2: Infraestructura de Hardware

- [Laboratorio 2.1: Implementación de un servidor virtual con RAID](tema2/lab-2.1-implementacion-de-un-servidor-virtual-con-raid.md)

- [Laboratorio 2.2: Simulación de un escenario de Failover y configuración de un sistema de Respaldo](tema2/lab-2.2-simulacion-de-un-escenario-de-failover-y-configuracion-de-un-sistema-de-respaldo.md)


### Tema 3: Infraestructura de Networking

- [Laboratorio 3.1: Proxy Inverso con Balanceador de Carga Avanzado y Servidores Web NGINX](tema3/lab-3.1-proxy-inverso-con-balanceador-de-carga-avanzado-y-servidores-web-nginx.md)

- [Laboratorio 3.2: Infraestructura de Red de una Organización con VLANs](tema3/lab-3.2-infraestructura-de-red-de-una-organizacion-con-vlans.md)


Crear una máquina virtual en VirtualBox para el servidor Proxy con las siguientes configuraciones:

Nombre y sistema operativo:
- Nombre: Lab3.1-Proxy
- Imagen ISO: ubuntu-24.04.3-live-server-amd64.iso

Hardware:
- Memoria base: 2048 MB
- Procesadores: 1 CPU

Disco duro:
- Disco Duro: 10,00 GB 

Red:
- Configuración de Red:
    - Adaptador 1: 
        - Habilitar adaptador de red:
            - Conectado a: NAT  

Instalar Ubuntu 24.04
- Seleccione el Idioma: Español
- Configuración del teclado: 
    - Disposición: Seleccione el idioma de su teclado
    - Variant: Seleccione la variante del teclado
- Choose the type of installation:
    - Marcar [X] Ubuntu Server
- Network configuration:
    - Dejar tal cual la configuración de la interfaz de red enp0s3 con DHCP, es muy probable que la IP asignada sea la 10.0.2.15/24
    - Configurar la interfaz enp0s8 con IPv4:
        - Método de IPv4: Manual
        - Subred: 192.168.10.0/29
        - Dirección: 192.168.10.1
        - Puerta de enlace: (vacío)
        - Servidores de nombres: (vacío)
        - Dominios de búsqueda: (vacío)
- [X] Instalar servidor OpenSSH


Crear una máquina virtual en VirtualBox para el primer servidor Servidor Web con las siguientes configuraciones:
Nombre y sistema operativo:
- Nombre: Lab3.1-WebServer1
- Imagen ISO: alpine-standard-3.22.1-x86_64.iso
- Tipo: Linux
- Subtype: Other Linux
- Versión: Other Linux (64-bit)

Hardware:
- Memoria base: 1024 MB
- Procesadores: 1 CPU

Disco Duro:
- Disco Duro: 5,00 GB 

Red:
- Configuración de Red:
    - Adaptador 1: 
        - Habilitar adaptador de red:
            - Conectado a: NAT (temporal, solo para descargar paquetes necesarios para instalar el SO)

Iniciar máquina virtual e instalar Alpine Linux:
- Una vez iniciado iniciar sesión con usuario root:
    localhost login: root (Enter)
- Ejecutar el instalador de Alpine Linux:
    localhost:~# setup-alpine (Enter)
- Seleccione la disposición del teclado:
    Select keyboard layout: [none] us (Enter)
- Seleccione la variante del teclado:
    Select variant (or 'abort'): us (Enter)
- Introduzca el hostname de la máquina virtual:
    Hostname
    --------
    Enter system hostname (fully qualified form, e.g. 'foo.example.org') [localhost] webserver1 (Enter)
- Seleccione la interfaz que tendrá que configurar:
    Interface
    ---------
    .....
    Which one do you want to initialize? (or '?' or 'done') [eth0] (Enter)
    
    IP address for eth0? (or 'dhcp', 'none' ?) [dhcp] (Enter)

    Do you want to do any manual network configuration? (y/n) [n] (Enter)
- Introduce la contraseña del usuario root:
    Root password
    -------------
    .....
    New password: ****** (Enter)
    Retype password: ****** (Enter)
- Selecciona la zona horaria:
    Timezone
    --------
    .....
    Which timezone are you in? (or '?' or 'none') [UTC] America/La_Paz (Enter)
- Configuración del Proxy (Internet):
    Proxy
    -----
    HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] (Enter)
- Configuración del servidor NTP:
    Network Time Protocol
    ---------------------
    .....
    Which NTP client to run? ('busybox', 'openntp', 'chrony' or 'none') [busybox] (Enter)
- Configuración de los repositorios de paquetes de Alpine Linux:
    APK Mirror
    ----------
    .....
    Enter mirror number or URL: [1] (Enter)
- Configuración del usuario alternativo a root:
     User 
    ------
    Setup a user? (enter a lower-case loginname, or 'no') [no] marcelo (Enter)
    .....
    Full name for user marcelo [marcelo] Marcelo Quispe Ortega (Enter)
    .....
    New password: ****** (Enter)
    Retype password: ****** (Enter)
    .....
    Enter ssh key or URL for marcelo (or 'none') [none] (Enter)
    .....
    Which ssh server? ('openssh', 'dropbear' or 'none') [openssh]
- Instalación en Disco duro:
    Disk & Install
    --------------
    Which disk(s) would you like to use? (or '?' for help or 'none') [none] sda (Enter)
    .....
    How would you like to use it? ('sys', 'data', 'crypt', 'lvm' or '?' for help) [?] sys (Enter)
    .....
    WARNING: Erase the above disk(s) and continue? (y/n) [n] y (Enter)
    .....
    Installation is complete. Please reboot.
    webserver1:~# poweroff (Enter)

- Seleccionamos la máquina virtual y hacemos clic en Configuración, luego en Almacenamiento, desmontamos del IDE Secundario (CD/DVD) la imagen del ISO de Alpine Linux y guardamos la configuración.

- Iniciamos nuevamente la Máquina virtual, iniciamos sesión como root e instalamos el paquetes nano:
    webserver:~# apk add nano (Enter)

- Apagamos la Máquina virtual:
    webserver:~# poweroff (Enter)

- Configuramos la Máquina virtual, cambiamos el Adaptador 1 de Red a:
    - Conectado a: "Red Interna"
    - Nombre: Lab3.1-SW

- Iniciamos nuevamente la Máquina virtual, accedemos como root y configuramos nuevamente la interfaz de red:
    webserver:~# nano /etc/network/interfaces (Enter)

    .....
    auto eth0
    iface eth0 inet static
        address 192.168.10.2
        netmask 255.255.255.248
        gateway 192.168.10.1

- Reiniciamos el servicio de red:
    webserver:~# /etc/init.d/networking restart

- Probamos la conexión con el gateway haciendo ping.
    webserver:~# ping 192.168.10.1


