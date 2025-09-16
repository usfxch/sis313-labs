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

    | Máquina Virtual | Departamento | VLAN ID | Subred |
    | ----------------|--------------|---------|--------|
    | `Router`        | -            | -       | 192.168.10.1/29 <br> 192.168.20.1/29 <br> 192.168.30.1/27 <br> 192.168.40.1/29 |
    | `Server-DMZ1`   | 

## 💻 Sección 2: Práctica guiada

### Paso 1: Instalación y Configuración de Red durante la instalación del S.O.


## ⚙️ Sección 3: Práctica en Grupo



### ✅ Evaluación del Laboratorio

La evaluación de este laboratorio se basará en los siguientes puntos, que demuestran el dominio de los conceptos y la correcta ejecución de los pasos.



4. **Informe de Laboratorio (30 pts)**

    El informe debe ser detallado con capturas de pantalla que demuestren cada uno de los pasos realizados.