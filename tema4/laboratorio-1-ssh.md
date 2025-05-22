# Laboratorio 4.1: Configuración de Acceso Remoto Seguro con SSH en Ubuntu Server 24.04 LTS

**Objetivo:** Configurar el servicio SSH en Ubuntu Server 24.04 LTS para permitir el acceso remoto seguro a la línea de comandos.

**Equipamiento Necesario:**

* Una máquina virtual o física con Ubuntu Server 24.04 LTS instalado.
* Otra máquina (con cualquier sistema operativo que tenga un cliente SSH instalado, como Linux, macOS o Windows con PowerShell/GitBash/PuTTY).
* Conectividad de red entre ambas máquinas.

**Pasos:**

**1. Instalación del Servidor SSH (si no está instalado):**

Ubuntu Server LTS suele tener el servidor SSH instalado por defecto. Sin embargo, si no es así, puedes instalarlo con el siguiente comando:

```bash
sudo apt update
```

```bash
sudo apt install openssh-server
```

**2. Verificación del Estado del Servicio SSH:**

Una vez instalado (o si ya estaba), verifica que el servicio SSH esté en ejecución:

```bash
sudo systemctl status ssh
```

La salida debería indicar que el servicio está active (running). Si no lo está, puedes iniciarlo con:

```bash
sudo systemctl start ssh
```

Y para que se inicie automáticamente en cada arranque:

```bash
sudo systemctl enable ssh
```

**3. Configuración del Servidor SSH (Archivo `sshd_config`):**

**3. Configuración del Servidor SSH (Archivo `sshd_config`):**

El archivo de configuración principal del servidor SSH es `/etc/ssh/sshd_config`. Ábrelo con un editor de texto con privilegios de superusuario:

```bash
sudo nano /etc/ssh/sshd_config
```

A continuación, se explican algunas directivas importantes dentro de este archivo y qué permiten realizar:

* **`Port 22`**: Define el puerto en el que el servidor SSH escucha las conexiones entrantes. El puerto predeterminado es el 22. Puedes cambiarlo por otro puerto no estándar por motivos de seguridad (ocultar el servicio de escaneos automáticos), pero recuerda que los clientes SSH deberán especificar el nuevo puerto.

* **`ListenAddress 0.0.0.0`**: Especifica las direcciones IP en las que el servidor SSH escucha. `0.0.0.0` significa escuchar en todas las interfaces disponibles. Puedes especificar una dirección IP específica si solo quieres permitir conexiones en una interfaz en particular.

* **`PermitRootLogin yes`**: Determina si se permite el inicio de sesión directamente como usuario `root`. **Por motivos de seguridad, se recomienda encarecidamente cambiar esto a `no`**. En su lugar, los usuarios deben iniciar sesión con sus cuentas regulares y luego usar `sudo` para realizar tareas administrativas.

* **`PubkeyAuthentication yes`**: Habilita la autenticación mediante claves públicas SSH. Esta es una forma más segura de autenticarse que el uso de contraseñas. Requiere la generación de un par de claves (pública y privada) y la copia de la clave pública al servidor.

* **`PasswordAuthentication yes`**: Habilita la autenticación mediante contraseñas. Si bien es más fácil de configurar inicialmente, es menos segura que la autenticación por clave pública. **Para entornos de producción, se recomienda deshabilitar esto (`no`) y usar solo la autenticación por clave pública.**

* **`PermitEmptyPasswords no`**: Impide que los usuarios con contraseñas vacías inicien sesión.

* **`ChallengeResponseAuthentication no`**: Controla la autenticación basada en desafío-respuesta (como la autenticación interactiva con contraseñas). Para la autenticación básica con contraseña, generalmente se mantiene en `yes`. Si planeas usar métodos más seguros como claves públicas, puedes considerar deshabilitarlo.

* **`UsePAM yes`**: Indica si SSH debe usar el sistema PAM (Pluggable Authentication Modules) para la autenticación. PAM proporciona una forma flexible de configurar diferentes métodos de autenticación.

* **`ClientAliveInterval 300`**: Envía una solicitud al cliente cada 300 segundos para verificar si sigue conectado.

* **`ClientAliveCountMax 3`**: Si el cliente no responde a las solicitudes `ClientAliveInterval` 3 veces consecutivas, el servidor cerrará la conexión. Estas directivas ayudan a detectar conexiones inactivas.

**Después de realizar cualquier cambio en el archivo `sshd_config`, debes guardar el archivo y reiniciar el servicio SSH para aplicar los cambios:**

```bash
sudo systemctl restart ssh
```

**4. Configuración del Firewall (UFW - Uncomplicated Firewall):**

Es importante configurar el firewall para permitir las conexiones entrantes al puerto SSH. Si UFW está habilitado en tu sistema, puedes permitir el tráfico SSH con el siguiente comando: 

```bash
sudo ufw allow ssh
```

Si cambiaste el puerto SSH en el archivo `/etc/ssh/sshd_config`, debes especificar el nuevo puerto en la regla del firewall. Por ejemplo, si tu nuevo puerto es 2222, usarías el comando:

```bash
sudo ufw allow 2222/tcp
```

Para verificar el estado del firewall y asegurarte de que la regla para SSH (o tu puerto personalizado) esté activa, puedes usar el comando:

```bash
sudo ufw status
```

**5. Conexión al Servidor SSH desde un Cliente:**

Desde tu máquina Host o anfitrión, utiliza un cliente SSH (PowerShell, GitBash o Putty) para conectarte al Ubuntu Server.

* **En Linux o macOS:** Abre la terminal y usa el comando:

    ```bash
    ssh usuario@<ip_del_servidor_ubuntu>
    ```

    Reemplaza `usuario` con un nombre de usuario válido en el Ubuntu Server y `<ip_del_servidor_ubuntu>` con la dirección IP de tu servidor Ubuntu. Si cambiaste el puerto, especifícalo con la opción `-p`:
    
    ```bash
    ssh -p <nuevo_puerto> usuario@<ip_del_servidor_ubuntu>
    ```

* **En Windows (con PuTTY):**
    1. Abre PuTTY.
    2. En el campo "Host Name (or IP address)", ingresa la dirección IP del Ubuntu Server.
    3. Asegúrate de que el "Port" sea 22 (o el puerto que configuraste).
    4. El "Connection type" debe ser "SSH".
    5. Haz clic en "Open".

    Se te pedirá la contraseña del usuario (si la autenticación por contraseña está habilitada) o se utilizará la clave privada si configuraste la autenticación por clave pública.

**6. Configuración de la Autenticación por Clave Pública:**

   La autenticación por clave pública es una forma más segura de acceder a tu servidor SSH que el uso de contraseñas. Implica la creación de un par de claves: una clave privada (que se guarda en tu máquina cliente) y una clave pública (que se copia al servidor).

   * **Generar el par de claves en tu máquina cliente:**

     Abre tu terminal (en Linux o macOS) o PuTTYgen (en Windows) y ejecuta el siguiente comando:

     ```bash
     ssh-keygen -t rsa -b 4096
     ```

     Se te pedirá que elijas una ubicación para guardar la clave (la predeterminada suele ser `~/.ssh/id_rsa` para la clave privada y `~/.ssh/id_rsa.pub` para la clave pública) y que ingreses una frase de contraseña (opcional, pero recomendada para mayor seguridad).

   * **Copiar la clave pública al servidor Ubuntu:**

     Hay varias formas de copiar la clave pública al servidor. Una de las más sencillas (si aún tienes acceso por contraseña) es usar `ssh-copy-id`:

     ```bash
     ssh-copy-id usuario@<ip_del_servidor_ubuntu>
     ```

     Se te pedirá tu contraseña en el servidor Ubuntu. Este comando copiará el contenido de tu archivo `~/.ssh/id_rsa.pub` (o la ubicación donde guardaste tu clave pública) al archivo `~/.ssh/authorized_keys` en el directorio home del `usuario` en el servidor.

     Si `ssh-copy-id` no está disponible, puedes copiar manualmente el contenido de tu archivo de clave pública (`~/.ssh/id_rsa.pub`) y añadirlo al archivo `~/.ssh/authorized_keys` en el servidor. Primero, conéctate al servidor por SSH (con contraseña) y luego ejecuta:

     ```bash
     mkdir -p ~/.ssh
     chmod 700 ~/.ssh
     nano ~/.ssh/authorized_keys
     ```

     Pega el contenido de tu clave pública en este archivo. Guarda y cierra el archivo. Luego, asegúrate de que los permisos del archivo `authorized_keys` sean correctos:

     ```bash
     chmod 600 ~/.ssh/authorized_keys
     ```

   * **Deshabilitar la autenticación por contraseña (opcional, pero recomendado para mayor seguridad):**

     Edita el archivo `/etc/ssh/sshd_config` en el servidor (como se explicó en el Paso 3) y cambia la directiva `PasswordAuthentication` a `no`:

     ```
     PasswordAuthentication no
     ```

     Guarda el archivo y reinicia el servicio SSH:

     ```bash
     sudo systemctl restart ssh
     ```

   Después de completar estos pasos, deberías poder conectarte a tu servidor Ubuntu por SSH desde tu máquina cliente sin necesidad de ingresar una contraseña (siempre y cuando la clave privada correspondiente esté presente en tu cliente y, si configuraste una frase de contraseña para la clave privada, la ingreses cuando sea necesario).

**7. Utilización de SCP (Secure Copy):**

   SCP es una utilidad de línea de comandos que permite la transferencia segura de archivos entre sistemas a través de una conexión SSH. Puedes usar SCP para copiar archivos desde tu máquina cliente al servidor Ubuntu o viceversa.

   * **Copiar un archivo desde tu máquina cliente al servidor Ubuntu:**

     ```bash
     scp /ruta/del/archivo_local usuario@<ip_del_servidor_ubuntu>:/ruta/de/destino_remoto
     ```

     Reemplaza `/ruta/del/archivo_local` con la ruta del archivo que quieres copiar en tu máquina cliente, `usuario` con tu nombre de usuario en el servidor Ubuntu, `<ip_del_servidor_ubuntu>` con la dirección IP del servidor, y `/ruta/de/destino_remoto` con la ubicación donde quieres guardar el archivo en el servidor. Si estás utilizando un puerto SSH no estándar, debes especificarlo con la opción `-P` (en mayúscula):

     ```bash
     scp -P <nuevo_puerto> /ruta/del/archivo_local usuario@<ip_del_servidor_ubuntu>:/ruta/de/destino_remoto
     ```

   * **Copiar un archivo desde el servidor Ubuntu a tu máquina cliente:**

     ```bash
     scp usuario@<ip_del_servidor_ubuntu>:/ruta/del/archivo_remoto /ruta/de/destino_local
     ```

     Reemplaza `/ruta/del/archivo_remoto` con la ruta del archivo que quieres copiar del servidor, `usuario` y `<ip_del_servidor_ubuntu>` con la información de conexión al servidor, y `/ruta/de/destino_local` con la ubicación donde quieres guardar el archivo en tu máquina cliente. Si estás utilizando un puerto SSH no estándar:

     ```bash
     scp -P <nuevo_puerto> usuario@<ip_del_servidor_ubuntu>:/ruta/del/archivo_remoto /ruta/de/destino_local
     ```

   SCP utiliza la misma autenticación que SSH. Si configuraste la autenticación por clave pública, no se te pedirá una contraseña al usar SCP. Si la autenticación por contraseña está habilitada, se te solicitará la contraseña del usuario.

**Práctica individual:**

1.  **Seguridad Primero:**
    * Edita el archivo `/etc/ssh/sshd_config`.
    * Cambia la configuración para no permitir acceso remoto al superadministrador.
    * Deshabilita la autenticación por contraseña.
    * Guarda los cambios y reinicia el servicio SSH.
    * **Importante:** Antes de cerrar la sesión actual, asegúrate de tener configurada la autenticación por clave pública para un usuario no root con permisos `sudo`, de lo contrario, podrías perder el acceso al servidor.

2.  **Cambio de Puerto:**
    * Elige un puerto no estándar (por ejemplo, un número entre 1025 y 65535).
    * Edita el archivo de configuración de SSH y cambia al puerto elegido.
    * Actualiza la configuración del firewall UFW para permitir el tráfico en el nuevo puerto (y si permitiste `ssh` anteriormente, puedes eliminar esa regla del UFW).
    * Reinicia el servicio SSH .
3.  **Acceso remoto**
    * Intenta conectarte al servidor SSH desde tu cliente utilizando el nuevo puerto.
    * Copia una carpeta de tu máquina Host hacia tu Ubuntu Server.
    * Copia un archivo de tu Ubuntu Server hacia tu máquina Host.

3.  **Investigación Adicional:**
    * Investiga y explica brevemente la función de las siguientes directivas en el archivo `/etc/ssh/sshd_config`:
        * `AllowUsers`
        * `AllowGroups`
        * `DenyUsers`
        * `DenyGroups`
    * Implementa una de estas directivas para restringir el acceso SSH a un usuario o grupo específico.