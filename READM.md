# 🚀 Laboratorio Integrado de Redes y Seguridad: VPN IPsec entre Cisco R1 y FortiGate con Servicio Web VIP (DNAT)

[![GNS3](https://img.shields.io/badge/Simulator-GNS3-blue.svg)](https://www.gns3.com/)
[![Fortinet](https://img.shields.io/badge/Firewall-FortiGate_v7.x-red.svg)](https://www.fortinet.com/)
[![Cisco](https://img.shields.io/badge/Router-Cisco_IOS-navy.svg)](https://www.cisco.com/)
[![Linux](https://img.shields.io/badge/OS-Ubuntu_Linux-orange.svg)](https://ubuntu.com/)

---

## 📹 Video Demostrativo

[![Ver Demostración del Laboratorio](https://img.youtube.com/vi/TU_VIDEO_ID/maxresdefault.jpg)](https://www.youtube.com/watch?v=TU_VIDEO_ID)

> 🎬 **Nota:** Haz clic en la imagen superior para ver el video completo de demostración en YouTube, donde se valida la conectividad Web externa (VIP), la negociación del túnel IPsec (Fase 1 y Fase 2) y el acceso SSH administrado desde el cliente Ubuntu.

---

## 📋 Tabla de Contenido

1. [Propósito del Laboratorio](#-propósito-del-laboratorio)
2. [Topología de Red](#-topología-de-red)
3. [Tabla de Direccionamiento](#-tabla-de-direccionamiento)
4. [Funcionamiento de la Configuración](#-funcionamiento-de-la-configuración)
5. [Configuraciones Implementadas](#-configuraciones-implementadas)
6. [Políticas de Seguridad](#-políticas-de-seguridad)
7. [Validación y Pruebas](#-validación-y-pruebas)
8. [Diagramas y Evidencias](#-diagramas-y-evidencias)
9. [Running Configurations](#-running-configurations)
10. [Scripts y Archivos Utilizados](#-scripts-y-archivos-utilizados)
11. [Conclusión](#-conclusión)

---

## 🎯 Propósito del Laboratorio

El propósito principal de este laboratorio es diseñar, implementar y validar un entorno de red híbrido e interconectado de manera segura en **GNS3**, integrando tecnologías de enrutamiento Cisco, seguridad perimetral Fortinet y clientes/servidores Linux. 

### Objetivos Específicos:
* **Acceso Público a Servicios Internos (DNAT/VIP):** Publicar un servidor Web local (`172.20.24.2`) a la red pública/WAN a través de un **Virtual IP (VIP)** en el FortiGate, permitiendo conexiones HTTP/HTTPS desde el exterior.
* **Interconexión Segura Sitio a Sitio (VPN IPsec):** Establecer un túnel encriptado IPsec (IKEv1) entre el Router **Cisco R1** y el Firewall **FortiGate**, garantizando la confidencialidad e integridad del tráfico administrativo.
* **Gestión Segura End-to-End:** Permitir a los clientes de la red corporativa (VLAN 10 / Ubuntu Client) acceder al servicio de gestión SSH (`Puerto 22`) del servidor remoto exclusivamente a través del túnel VPN encriptado.

---

## 🌐 Topología de Red

```text
               +--------------------------------------------------+
               |                  RED WAN (INTERNET)               |
               |                     202.40.88.0/30               |
               +------------------------+-------------------------+
                                        |
                   202.40.88.2/30       |       202.40.88.1/30
                +-----------------------+-----------------------+
                |                                               |
         [FastEthernet1/0]                                   [port1]
          +-----------+                                   +-----------+
          |  Cisco    |                                   | FortiGate |
          | Router R1 |=== === === TÚNEL IPsec === === ===| Firewall  |
          +-----------+                                   +-----------+
         [FastEthernet0/0]                                   [port3]
            192.168.86.1/25                                 172.20.24.1/28
                |                                               |
                |                                               |
     +----------+----------+                         +----------+----------+
     |   VLAN 10 (LAN)     |                         |   DMZ / Servidores  |
     |   192.168.86.0/25   |                         |   172.20.24.0/28    |
     +----------+----------+                         +----------+----------+
                |                                               |
         192.168.86.4/25                                 172.20.24.2/28
          +-----------+                                   +-----------+
          |  Ubuntu   |                                   |    WEB    |
          |  Client   |                                   |  SERVER   |
          +-----------+                                   +-----------+
