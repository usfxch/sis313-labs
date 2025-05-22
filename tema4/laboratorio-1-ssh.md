# Laboratorio 1: Configuración de Acceso Remoto Seguro con SSH en Ubuntu Server 24.04 LTS

**Objetivo:** Configurar el servicio SSH en Ubuntu Server 24.04 LTS para permitir el acceso remoto seguro a la línea de comandos.

**Equipamiento Necesario:**

* Una máquina virtual o física con Ubuntu Server LTS instalado.
* Otra máquina (con cualquier sistema operativo que tenga un cliente SSH instalado, como Linux, macOS o Windows con PuTTY).
* Conectividad de red entre ambas máquinas.

**Pasos:**

**1. Instalación del Servidor SSH (si no está instalado):**

Ubuntu Server LTS suele tener el servidor SSH instalado por defecto. Sin embargo, si no es así, puedes instalarlo con el siguiente comando:

```bash
sudo apt update
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