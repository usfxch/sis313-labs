# Laboratorio 3.2: Infraestructura de Red de una Organización con VLANs

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 2/2025

## 🎯 Objetivo del Laboratorio

El objetivo de este laboratorio es que los estudiantes sean capaces de:

- **Diseñar e implementar una arquitectura de red empresarial con VLANs** para segmentar los departamentos de una organización.

- **Configurar un router con Linux** para que gestione el enrutamiento inter-VLAN, el acceso a internet y las políticas de seguridad.

- **Aplicar reglas de firewall (UFW)** para controlar el flujo de tráfico entre las diferentes VLANs y la red externa.

- **Comprender y configurar interfaces** `trunk` y etiquetado de VLANs en máquinas virtuales.

- **Demostrar el funcionamiento de las políticas de acceso** establecidas entre los diferentes departamentos.

## 🛠️ Sección 1: Preparación del Entorno Virtual

En esta sección, se configurará la infraestructura con las siguientes máquinas virtuales y su correspondiente esquema de red. Se recomienda el uso de VirtualBox por su facilidad para configurar interfaces de red.

1. **Máquina Virtual: Router (Ubuntu Server 24.04)**

    - **Interfaz 1 (`enp0s3`)**: NAT. Se conecta a la red del anfitrión para acceso a internet.

    - **Interfaz 2 (`enp0s8`)**: Red Interna (Tipo "Red Interna" o "Host-only"). Esta interfaz se configurará como un `trunk` para transportar las VLANs.

2. **Máquinas Virtuales: PC TI, PC Ventas, PC Contabilidad, Servers DMZ (Alpine Linux)**

    - Cada una de estas VMs tendrá una única interfaz de red conectada a la misma "Red Interna" que la interfaz `enp0s8` del Router.

    - **Importante:** En la configuración de la red de tu hipervisor, asegúrate de que la "Red Interna" esté configurada como un switch que permite el paso de tramas etiquetadas (VLAN-aware).

**Esquema de la Infraestructura**

| Máquina Virtual | Departamento | VLAN ID | Subred |
| ----------------|--------------|---------|--------|
| `Router`        | -            | -       | 192.168.10.1/29 <br> 192.168.20.1/29 <br> 192.168.30.1/27 <br> 192.168.40.1/29 |
| `Server-DMZ1`   | DMZ          | 10      | 192.168.10.2/29 |
| `Server-DMZ2`	  | DMZ	         | 10	   | 192.168.10.3/29 |
| `PC-TI`         | TI           | 20      | 192.168.20.2/29 |
| `PC-Ventas`     | Ventas       | 30      | 192.168.30.2/27 |
| `PC-Contabilidad`| Contabilidad       | 40      | 192.168.40.2/29 |

## 💻 Sección 2: Práctica guiada

### Paso 1: Configuración de VLANs y Router

- **En la VM Router (Ubuntu):**

    - Instala las herramientas necesarias para trabajar con VLANs:

        ```bash
        sudo apt install vlan
        ```

    - Carga el módulo del kernel para VLANs: 

        ```bash
        sudo modprobe 8021q
        ```

    - Configura las interfaces de red VLAN. Para el enrutamiento inter-VLAN, cada sub-interfaz tendrá un gateway que será su propia dirección IP.

    - Edita el archivo de configuración de red para crear las sub-interfaces virtuales (ej. `vlan10`, `vlan20`, etc.) y asignarles las IPs estáticas de cada VLAN.

        ```bash
        nano /etc/netplan/50-cloud-init.yaml
        ```

        ```bash
        network:
          version: 2
          ethernets:
            enp0s3:
              dhcp4: true
            enp0s8:
              dhcp4: no
              optional: true
          vlans:
            vlan10:
              link: enp0s8
              id: 10
              addresses:
                - 192.168.10.1/29
              nameservers:
                addresses: [8.8.8.8]
            vlan20:
              link: enp0s8
              id: 20
              addresses:
                - 192.168.20.1/29
              nameservers:
                addresses: [8.8.8.8]
            vlan30:
              link: enp0s8
              id: 30
              addresses:
                - 192.168.30.1/27
              nameservers:
                addresses: [8.8.8.8]
            vlan40:
              link: enp0s8
              id: 40
              addresses:
                - 192.168.40.1/29
              nameservers:
                addresses: [8.8.8.8]
        ```

- **Enrutamiento y NAT:**

    - Habilitar el reenvío de paquetes en el kernel

        Edita el archivo `sysctl.conf`:
        ```bash
        sudo nano /etc/sysctl.conf
        ```
    
        Asegúrate de que esta línea esté presente y no comentada:
        ```bash
        net.ipv4.ip_forward=1
        ```

        Aplicar los cambios inmediatamente
        ```bash
        sudo sysctl -p
        ```

### Paso 2: Configuración de la VM de Contabilidad

- **En la VM de Contabilidad (Alpine):**

    - Instala la herramienta `vlan`:
    
        ```bash
        apk add vlan
        ```

    - Configura la interfaz de red con la VLAN 40.

        ```bash
        nano /etc/network/interfaces
        ```

        ```nano
        auto lo
        iface lo inet loopback

        auto eth0.40
        iface eth0.40 inet static
            address 192.168.40.2
            netmask 255.255.255.248
            gateway 192.168.40.1
            vlan-id 40

        auto eth0
        iface eth0 inet manual
            up ip link set $IFACE up
            down ip link set $IFACE down
        ```

        > Asigna la IP estática `192.168.40.2` con la máscara `/29`.

        > Configura el gateway a la IP del router en esa VLAN: `192.168.40.1`.

    - **Prueba:** Verifica la conexión con el router (`ping 192.168.40.1`) y el acceso a internet (`ping google.com`).

### Paso 3: Configuración de la VM de Ventas

- **En la VM de Ventas (Alpine):**

    - Instala la herramienta `vlan`:

        ```bash
        apk add vlan
        ```

    - Configura la interfaz de red con la VLAN 30.

        ```bash
        nano /etc/network/interfaces
        ```

        ```nano
        auto lo
            iface lo inet loopback

        auto eth0.30
        iface eth0.30 inet static
            address 192.168.30.2
            netmask 255.255.255.224
            gateway 192.168.30.1
            vlan-id 30

        auto eth0
        iface eth0 inet manual
            up ip link set $IFACE up
            down ip link set $IFACE down
        ```

        > Asigna la IP estática `192.168.30.2` con la máscara `/27`.

        > Configura el gateway a la IP del router: `192.168.30.1`.

    - **Prueba:** Verifica la conexión con el router (`ping 192.168.30.1`) pero confirma que **no tiene acceso a internet**.

### Paso 4: Configuración de UFW en el Router

- **En la VM Router (Ubuntu):**

    - Instala y habilita `ufw`: 
    
        ```bash
        sudo apt install ufw
        ```

    - Habilita `ssh` en `ufw`: 
    
        ```bash
        sudo ufw allow ssh
        ```

    - Habilita `ufw` desde el arranque: 
    
        ```bash
        sudo ufw enable
        ```

    - Configura las siguientes reglas para controlar el tráfico entre las VLANs. Es crucial establecer las reglas en el orden correcto.

        Permiso para TI (VLAN 20) a todos
        ```bash
        sudo ufw route allow in on vlan20 out on vlan10
        sudo ufw route allow in on vlan20 out on vlan30
        sudo ufw route allow in on vlan20 out on vlan40
        ```

        Permiso para Ventas (VLAN 30) a DMZ
        ```bash
        sudo ufw route allow in on vlan30 out on vlan10
        ```

        Permiso para Contabilidad (VLAN 40) a DMZ y Ventas
        ```bash
        sudo ufw route allow in on vlan40 out on vlan10
        sudo ufw route allow in on vlan40 out on vlan30
        ```

        Denegar acceso de DMZ (VLAN 10) a todos
        ```bash
        sudo ufw route deny in on vlan10 out on vlan20
        sudo ufw route deny in on vlan10 out on vlan30
        sudo ufw route deny in on vlan10 out on vlan40
        ```

        Denegar acceso de Ventas (VLAN 30) a TI y Contabilidad
        ```bash
        sudo ufw route deny in on vlan30 out on vlan20
        sudo ufw route deny in on vlan30 out on vlan40
        ```

        Denegar acceso de Contabilidad (VLAN 40) a TI
        ```bash
        sudo ufw route deny in on vlan40 out on vlan20
        ```

    - Configura las siguientes reglas para el acceso a Internet desde TI y Contabilidad.

        ```bash
        sudo nano /etc/ufw/before.rules
        ```

        Añade al inicio del archivo la siguiente configuración, que son reglas de enmascaramiento para que las VLANs tengan acceso a Internet:
        ```nano
        *nat
        :POSTROUTING ACCEPT [0:0]
        -A POSTROUTING -s 192.168.20.0/24 -o enp0s3 -j MASQUERADE
        -A POSTROUTING -s 192.168.40.0/24 -o enp0s3 -j MASQUERADE
        COMMIT
        ```

    - **Pruebas:** Desde la VM de Ventas, intenta conectar por `ssh` a la PC Contabilidad y a un Servidor de la DMZ para verificar si tienes o no acceso.

## ⚙️ Sección 3: Práctica en Grupo

Se requiere que el grupo complete la configuración de la infraestructura en una única PC anfitriona con mayores recursos. Los miembros del grupo que no tengan acceso a esta PC pueden seguir el mismo proceso en sus equipos personales, replicando la configuración hasta donde sus recursos lo permitan.

El objetivo es que el o los estudiantes completen la configuración de las **VLANs 10 y 20 (DMZ y TI)** y todas las reglas de UFW correspondientes para cumplir con las restricciones establecidas:

- **VMs DMZ (VLAN 10)**: Deben estar configuradas con sus IPs estáticas. Su acceso a internet y a las otras VLANs debe estar denegado por defecto.

- **VM de TI (VLAN 20)**: Debe estar configurada con su IP estática. Debe tener acceso a internet y a todas las otras VLANs (DMZ, Ventas y Contabilidad).

Al final del ejercicio, el grupo deberá demostrar a través de pruebas de acceso (ej. `ssh`) que todas las políticas de acceso y denegación están funcionando correctamente.

### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se centrará en la correcta implementación de la arquitectura y el funcionamiento de las políticas de seguridad. Se valorará la habilidad para resolver problemas y la comprensión de los conceptos clave.

1. **Configuración del Entorno y Enrutamiento (10 pts)**

    - **Configuración del Router:** Demuestra que las interfaces VLAN y el enrutamiento inter-VLAN están correctamente configurados en el router de Ubuntu.

    - **Conectividad de las VLANs:** Confirma que las máquinas virtuales de **Contabilidad** y **Ventas** están en sus respectivas VLANs, que tienen las IPs estáticas y el gateway correctos.

    - **Acceso a Internet:** Verifica que la VLAN de Contabilidad tiene acceso a internet y que la VLAN de Ventas no lo tiene, tal como lo exige el problema.

2. **Implementación de Políticas de Acceso (20 pts)**

    - **Reglas de UFW:** Presenta la configuración de UFW en el router y explica cómo cada regla permite o deniega el tráfico entre las VLANs.

    - **Pruebas de Conectividad:**

        - Demuestra que la VLAN de Contabilidad puede acceder a la VLAN de Ventas y a la DMZ.

        - Demuestra que la VLAN de Ventas solo puede acceder a la DMZ.

        - Confirma que la VLAN de la DMZ no tiene acceso a las otras VLANs ni a internet.

3. **Práctica en Grupo y Demostración (40 pts)**

    - **Configuración de las VLANs Restantes:** El grupo debe mostrar que las VLANs **10 (DMZ)** y **20 (TI)** están correctamente configuradas, incluyendo sus IPs estáticas y el enrutamiento.

    - **Aplicación de Políticas:** Demuestra que las políticas de acceso para TI (acceso a todos los departamentos y a internet) y para la DMZ (sin acceso saliente) funcionan como se espera.

    - **Colaboración y Presentación:** La evaluación considerará la calidad del trabajo en equipo, la claridad de la presentación y la capacidad para explicar la configuración y las pruebas realizadas.

4. **Informe de Laboratorio (30 pts)**

    El informe debe ser detallado con capturas de pantalla que demuestren cada uno de los pasos realizados.