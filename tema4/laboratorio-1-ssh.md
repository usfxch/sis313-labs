# Laboratorio 4.1: Configuración de Acceso Remoto Seguro con SSH en Ubuntu Server 24.04 LTS

**Objetivo:** Configurar el servicio SSH en Ubuntu Server 24.04 LTS para permitir el acceso remoto seguro a la línea de comandos.

**Equipamiento Necesario:**

* Una máquina virtual o física con Ubuntu Server 24.04 LTS instalado.
* Otra máquina (con cualquier sistema operativo que tenga un cliente SSH instalado, como Linux, macOS o Windows con PuTTY).
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

Desde tu máquina host o anfitrión, utiliza un cliente SSH (PowerShell, GitBash o Putty) para conectarte al Ubuntu Server.

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

**Ejercicio Práctico:**

1.  **Seguridad Primero:**
    * Edita el archivo `/etc/ssh/sshd_config`.
    * Cambia la directiva `PermitRootLogin` a `no`.
    * Deshabilita la autenticación por contraseña (`PasswordAuthentication no`).
    * Guarda los cambios y reinicia el servicio SSH (`sudo systemctl restart ssh`).
    * **Importante:** Antes de cerrar la sesión actual, asegúrate de tener configurada la autenticación por clave pública para un usuario no root con permisos `sudo`, de lo contrario, podrías perder el acceso al servidor.

2.  **Cambio de Puerto:**
    * Elige un puerto no estándar (por ejemplo, un número entre 1025 y 65535).
    * Edita `/etc/ssh/sshd_config` y cambia la directiva `Port` al puerto elegido.
    * Actualiza la configuración del firewall UFW para permitir el tráfico en el nuevo puerto (`sudo ufw allow <nuevo_puerto>/tcp`, y si permitiste `ssh` anteriormente, puedes eliminar esa regla con `sudo ufw delete allow ssh`).
    * Reinicia el servicio SSH (`sudo systemctl restart ssh`).
    * Intenta conectarte al servidor SSH desde tu cliente utilizando el nuevo puerto (recuerda usar la opción `-p` en el comando `ssh` o especificar el puerto en PuTTY).

3.  **Investigación Adicional:**
    * Investiga y explica brevemente la función de las siguientes directivas en el archivo `/etc/ssh/sshd_config`:
        * `AllowUsers`
        * `AllowGroups`
        * `DenyUsers`
        * `DenyGroups`
    * Implementa una de estas directivas para restringir el acceso SSH a un usuario o grupo específico.
