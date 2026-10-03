# 🚀 Laboratorio Integrado de Redes y Seguridad: VPN IPsec entre Cisco R1 y FortiGate con Servicio Web VIP (DNAT)

[![GNS3](https://img.shields.io/badge/Simulator-GNS3-blue.svg)](https://www.gns3.com/)
[![Fortinet](https://img.shields.io/badge/Firewall-FortiGate_v7.x-red.svg)](https://www.fortinet.com/)
[![Cisco](https://img.shields.io/badge/Router-Cisco_IOS-navy.svg)](https://www.cisco.com/)
[![Linux](https://img.shields.io/badge/OS-Ubuntu_Linux-orange.svg)](https://ubuntu.com/)

---

## 📹 Video Demostrativo

[![Ver Demostración del Laboratorio](https://img.youtube.com/vi/TU_VIDEO_ID/maxresdefault.jpg)](https://www.youtube.com/watch?v=TU_VIDEO_ID)

> 🎬 **Nota:** Haz clic en la imagen superior para ver la demostración en video del laboratorio. En el video se valida el flujo de acceso Web externo a través de la Virtual IP (DNAT), la negociación del túnel IPsec (Fase 1 y Fase 2 con PFS Group 14) entre Cisco R1 y FortiGate, y el acceso administrativo cifrado mediante SSH desde la PC cliente Ubuntu.

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

El propósito fundamental de este laboratorio es diseñar, desplegar y validar un entorno de red empresarial híbrido simulado en **GNS3**, que integra tecnologías de enrutamiento **Cisco IOS**, seguridad perimetral **Fortinet FortiGate** y sistemas operativos **Linux (Ubuntu)**. 

El escenario resuelve dos requerimientos críticos de diseño en infraestructura corporativa:

1. **Publicación Segura de Servicios Internos (DNAT / VIP):** Permitir que usuarios externos accedan a un servidor web ubicado en la DMZ interna (`172.20.24.2`) mediante la dirección IP pública WAN del firewall (`202.40.88.1`), protegiendo el direccionamiento privado real de la infraestructura.
2. **Interconexión Segura Sitio a Sitio y Gestión Cifrada (VPN IPsec Site-to-Site):** Establecer un túnel encriptado IPsec (IKEv1) entre el Router de la sede cliente (**Cisco R1**) y el Firewall de la sede principal (**FortiGate**). Esto permite que los administradores ubicados en la VLAN 10 (`192.168.86.0/25`) gestionen de forma segura el servidor mediante **SSH (`puerto 22`)**, garantizando confidencialidad, integridad y autenticidad del tráfico a través de un canal público no seguro.

---

## 🌐 Topología de Red

```text
               +----------------------------------------------------+
               |                 RED WAN (INTERNET)                 |
               |                   202.40.88.0/30                   |
               +-------------------------+--------------------------+
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
```

---

## 📊 Tabla de Direccionamiento

| Dispositivo | Interfaz | Dirección IP / Máscara | Gateway Predeterminado | Propósito / Zona de Red |
| :--- | :--- | :--- | :--- | :--- |
| **Cisco R1** | `FastEthernet1/0` | `202.40.88.2/30` | N/A | Interfaz WAN (Peer IPsec hacia FortiGate) |
| **Cisco R1** | `FastEthernet0/0` | `192.168.86.1/25` | N/A | Gateway de la VLAN 10 (Red Local de Usuarios) |
| **FortiGate** | `port1` | `202.40.88.1/30` | N/A | Interfaz WAN pública / VIP / Crypto Peer |
| **FortiGate** | `port3` | `172.20.24.1/28` | N/A | Gateway de la Red DMZ (Servidores) |
| **Ubuntu Client**| `ens3` | `192.168.86.4/25` | `192.168.86.1` | Estación de trabajo cliente (VLAN 10) |
| **WEB-SERVER** | `ens3` | `172.20.24.2/28` | `172.20.24.1` | Servidor Linux (Servicios HTTP puerto 80 y SSH puerto 22) |

---

## ⚙️ Funcionamiento de la Configuración

### 1. Publicación de Servicios mediante Virtual IP (DNAT)
* **Recepción:** El tráfico proveniente del exterior con destino a la IP pública `202.40.88.1` en los puertos 80 (HTTP) o 443 (HTTPS) es capturado por la interfaz `port1` del FortiGate.
* **Traducción:** El motor de Virtual IP (**DNAT**) traduce la dirección de destino `202.40.88.1` hacia la dirección privada interna `172.20.24.2`.
* **Filtrado:** La política de firewall `VIP_WEB_ACCESS` evalúa y aprueba el paso del paquete desde la zona WAN (`port1`) hacia la zona DMZ (`port3`).

### 2. Túnel IPsec Site-to-Site (IKEv1)
* **Fase 1 (ISAKMP - Negociación de Canal Seguro):**
  * **Autenticación:** Clave compartida previa (*Pre-Shared Key*): `ClaveSegura2024`.
  * **Algoritmos Criptográficos:** Cifrado `DES`, Hashing `SHA-256`, Grupo Diffie-Hellman **`Group 14`** (2048-bit MODP).
  * **Propósito:** Autenticar mutuamente a Cisco R1 y FortiGate y establecer un canal seguro IKE SA.
* **Fase 2 (IPsec SA - Selección de Tráfico de Interés):**
  * **Selectores de Tráfico:**
    * **Red Local (FortiGate):** `172.20.24.0/28` (Subred Servidores)
    * **Red Remota (Cisco R1):** `192.168.86.0/25` (Subred Usuarios VLAN 10)
  * **PFS (Perfect Forward Secrecy):** Activado obligatoriamente con **`DH Group 14`**, garantizando que el compromiso de una clave no exponga sesiones pasadas.
* **Enrutamiento del Túnel:**
  * En **Cisco R1**, una ruta estática dirige el tráfico hacia `172.20.24.0/28` vía la interfaz WAN `FastEthernet1/0` apuntando a `202.40.88.1`. El *Crypto Map* intercepta el paquete al hacer *match* con la `access-list 101` y lo encapsula en ESP.
  * En **FortiGate**, una ruta estática dirige la subred `192.168.86.0/25` directamente a la interfaz virtual de túnel **`VPN_CISCO`**.

---

## 🛠️ Configuraciones Implementadas

### A. Cisco Router (R1) - Configuración Completa

```text
! Configuration for Cisco R1 Router
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
! Configuración de Interfaces
interface FastEthernet0/0
 description ** Gateway LAN VLAN 10 **
 ip address 192.168.86.1 255.255.255.128
 duplex auto
 speed auto
 no shutdown
!
interface FastEthernet1/0
 description ** Interfaz WAN conectada a FortiGate port1 **
 ip address 202.40.88.2 255.255.255.252
 crypto map CM_FORTI
 duplex auto
 speed auto
 no shutdown
!
! Configuración VPN IPsec - Fase 1 (ISAKMP)
crypto isakmp policy 10
 encr des
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
exit

crypto isakmp key ClaveSegura2024 address 202.40.88.1

! Configuración VPN IPsec - Fase 2 & Crypto Map
crypto ipsec transform-set TS_VPN esp-des esp-sha-hmac
 mode tunnel
exit

crypto map CM_FORTI 10 ipsec-isakmp
 set peer 202.40.88.1
 set transform-set TS_VPN
 set pfs group14
 match address 101
exit

! Lista de Control de Acceso (Tráfico de Interés para la VPN)
access-list 101 permit ip 192.168.86.0 0.0.0.127 172.20.24.0 0.0.0.15

! Ruta Estática hacia la Red del Servidor Web
ip route 172.20.24.0 255.255.255.240 FastEthernet1/0 202.40.88.1

end
write memory
```

---

### B. FortiGate Firewall - Configuración CLI (Fase 1, Fase 2, VIP y Rutas)

```text
# 1. Configuración de Interfaces
config system interface
    edit "port1"
        set vdom "root"
        set ip 202.40.88.1 255.255.255.252
        set allowaccess ping https ssh
        set alias "WAN"
    next
    edit "port3"
        set vdom "root"
        set ip 172.20.24.1 255.255.255.240
        set allowaccess ping https ssh
        set alias "DMZ_SERVERS"
    next
end

# 2. Configuración de Virtual IP (DNAT)
config firewall vip
    edit "VIP_WEB_SERVER"
        set extip 202.40.88.1
        set mappedip "172.20.24.2"
        set extintf "port1"
        set portforward enable
        set protocol tcp
        set extport 80
        set mappedport 80
    next
end

# 3. Configuración del Túnel IPsec (Fase 1 y Fase 2)
config vpn ipsec phase1-interface
    edit "VPN_CISCO"
        set interface "port1"
        set peertype any
        set net-device disable
        set proposal des-sha256 des-sha1
        set dhgrp 14
        set remote-gw 202.40.88.2
        set psksecret ClaveSegura2024
    next
end

config vpn ipsec phase2-interface
    edit "VPN_CISCO_p2"
        set phase1name "VPN_CISCO"
        set proposal des-sha1 des-sha256
        set dhgrp 14
        set src-subnet 172.20.24.0 255.255.255.240
        set dst-subnet 192.168.86.0 255.255.255.128
    next
end

# 4. Ruta Estática apuntando al Túnel IPsec
config router static
    edit 0
        set dst 192.168.86.0 255.255.255.128
        set device "VPN_CISCO"
    next
end
```

---

## 🔒 Políticas de Seguridad

### Políticas de Firewall Implementadas en FortiGate:

```text
config firewall policy
    # Política 1: Permitir Acceso Web Público a través de Virtual IP
    edit 1
        set name "VIP_WEB_ACCESS"
        set srcintf "port1"
        set dstintf "port3"
        set action accept
        set srcaddr "all"
        set dstaddr "VIP_WEB_SERVER"
        set schedule "always"
        set service "HTTP" "HTTPS"
        set utm-status disable
    next

    # Política 2: Permitir Acceso SSH y PING Entrante desde la VPN
    edit 2
        set name "Allow_SSH_VPN"
        set srcintf "VPN_CISCO"
        set dstintf "port3"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "SSH" "PING"
        set utm-status disable
    next
end
```

---

## 🧪 Validación y Pruebas

### 1. Prueba de Acceso Web Directo desde Ubuntu vía VIP (DNAT)
Desde la terminal del cliente **Ubuntu** (`192.168.86.4`), se ejecuta el comando `curl`:

```bash
ubuntu@ubuntu-cloud:~$ curl --compressed http://202.40.88.1
```

**Resultado obtenido:**
```text
HTTP/1.1 200 OK
Content-Type: text/html
Content-Encoding: gzip
Date: Sat, 03 Oct 2026 02:05:04 GMT

<!DOCTYPE html>
<html>
<head><title>Servidor Web DMZ</title></head>
<body><h1>Acceso Exitoso a traves de FortiGate VIP (DNAT)</h1></body>
</html>
```

---

### 2. Verificación del Estado de la VPN IPsec en Cisco R1

Se valida que la Fase 1 esté en estado activo (**`QM_IDLE`**):

```text
R1# show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status
202.40.88.1     202.40.88.2     QM_IDLE              1    0 ACTIVE
```

Se valida la Fase 2 (SAs de IPsec), verificando que los contadores de paquetes cifrados (`encaps`) y descifrados (`decaps`) sean simétricos y mayores a 0, confirmando el flujo bidireccional:

```text
R1# show crypto ipsec sa

interface: FastEthernet1/0
    Crypto map tag: CM_FORTI, local addr 202.40.88.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.86.0/255.255.255.128/0/0)
   remote ident (addr/mask/prot/port): (172.20.24.0/255.255.255.240/0/0)
   current_peer 202.40.88.1 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 28, #pkts encrypt: 28, #pkts digest: 28
    #pkts decaps: 28, #pkts decrypt: 28, #pkts verify: 28
    #send errors 0, #recv errors 0

     local crypto endpt.: 202.40.88.2, remote crypto endpt.: 202.40.88.1
     PFS (Y/N): Y, DH group: group14
```

---

### 3. Conexión SSH Administrada a través de la VPN

Desde el cliente **Ubuntu** (`192.168.86.4`), se establece una sesión SSH cifrada directamente a la IP privada del servidor (`172.20.24.2`):

```bash
ubuntu@ubuntu-cloud:~$ ssh ubuntu@172.20.24.2
ubuntu@172.20.24.2's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)
* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

Last login: Sat Oct  3 02:10:12 2026 from 192.168.86.4
ubuntu@WEB-SERVER:~$ 
```
> **Confirmación:** La sesión SSH inicia correctamente sobre el túnel IPsec encriptado.

---

## 📸 Diagramas y Evidencias

| Descripción de la Evidencia | Captura / Imagen |
| :--- | :--- |
| **Página Web recibida mediante VIP (curl)** | ![Prueba HTTP VIP](docs/images/curl_web_vip.png) |
| **Verificación ISAKMP y IPsec SA en R1** | ![Estado IPsec R1](docs/images/cisco_ipsec_sa.png) |
| **Sesión SSH Remota Establecida** | ![SSH sobre VPN](docs/images/ssh_vpn_success.png) |
| **Panel de Estado IPsec Tunnels en FortiGate GUI** | ![FortiGate Dashboard](docs/images/fortigate_vpn_dashboard.png) |

---

## 📜 Running Configurations

Las configuraciones completas descargadas directamente de los equipos se encuentran archivadas en la carpeta `/configs`:

* 📄 **Cisco Router R1:** [`configs/R1_running_config.txt`](configs/R1_running_config.txt)
* 📄 **FortiGate Firewall:** [`configs/FortiGate_config.conf`](configs/FortiGate_config.conf)

---

## 📁 Scripts y Archivos Utilizados

En el directorio `/scripts` del repositorio se incluyen las herramientas de automatización utilizadas durante la fase de despliegue:

* 🐍 **`scripts/start_web_server.py`**: Script en Python para inicializar un servidor de pruebas en el puerto 80 dentro del WEB-SERVER.
* 📜 **`scripts/network_setup_ubuntu.sh`**: Script en Bash para configurar las tarjetas de red, IP estática y rutas de la máquina Ubuntu.

---

## 🏁 Conclusión

El desarrollo de este laboratorio permitió validar la integración exitosa entre plataformas heterogéneas (**Cisco IOS** y **Fortinet FortiOS**). Se lograron los objetivos planteados mediante la implementación de:

1. **Traducción de Direcciones (DNAT/VIP):** Permitió la exposición segura del servicio Web sin vulnerar la topología interna.
2. **Sincronización Criptográfica Rigurosa:** La alineación exacta de los parámetros de Fase 1, Fase 2 y **PFS Group 14** garantizó la estabilidad del túnel IPsec, resolviendo los problemas de paquetes unidireccionales (`decaps: 0`) mediante la correcta asignación de la ruta estática apuntando a la interfaz de túnel `VPN_CISCO` en el FortiGate.
