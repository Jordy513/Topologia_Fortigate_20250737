# FortiGate — Topología de Seguridad con Políticas Avanzadas

### Jordy Jose Rosario Ortiz · Matrícula: 2025-0737

**Seguridad de Redes 2026-C-2 · ITLA**

---

## 📋 Tabla de Contenido

1. [Objetivo de la Red](#1-objetivo-de-la-red)
2. [Topología y Direccionamiento](#2-topología-y-direccionamiento)
   - [Diagrama de Topología](#21-diagrama-de-topología)
   - [Tabla de Interfaces](#22-tabla-de-interfaces)
3. [Configuraciones por la GUI](#3-configuraciones-por-la-gui)
   - [3.0 Acceso Inicial — Configuración por CLI](#30-acceso-inicial--configuración-por-cli)
   - [3.1 Configuración de Interfaces](#31-configuración-de-interfaces)
   - [3.2 Servidor DHCP en LAN de Usuarios](#32-servidor-dhcp-en-lan-de-usuarios)
   - [3.3 Ruta por Defecto](#33-ruta-por-defecto)
   - [3.4 NAT — Acceso a Internet](#34-nat--acceso-a-internet)
   - [3.5 Política HTTP — LAN Usuarios a LAN Servidores](#35-política-http--lan-usuarios-a-lan-servidores)
   - [3.6 Bloqueo de Redes Sociales](#36-bloqueo-de-redes-sociales)
   - [3.7 Bloqueo de Llamadas por WhatsApp](#37-bloqueo-de-llamadas-por-whatsapp)
   - [3.8 Bloqueo de itla.edu.do](#38-bloqueo-de-itlaeduedo)
   - [3.9 Detección y Bloqueo de Escaners de Red](#39-detección-y-bloqueo-de-escaners-de-red)
   - [3.10 WAF — Protección del Servidor Web](#310-waf--protección-del-servidor-web)
4. [Capturas de Pantalla](#4-capturas-de-pantalla)
5. [Video Demostrativo](#5-video-demostrativo)
6. [Referencias](#6-referencias)

---

## 1. Objetivo de la Red

Esta topología implementa un **FortiGate como firewall perimetral y de segmentación interna** para una red corporativa con dos segmentos diferenciados: una LAN de usuarios con acceso a Internet controlado y una LAN de servidores con acceso restringido únicamente a HTTP.

Los objetivos específicos de seguridad implementados son:

* Proveer acceso a Internet a los usuarios mediante NAT, con asignación dinámica de IPs por DHCP.
* Restringir la comunicación entre la LAN de usuarios y la LAN de servidores exclusivamente al protocolo HTTP (puerto 80), bloqueando cualquier otro tráfico entre ambos segmentos.
* Bloquear el acceso a redes sociales desde la LAN de usuarios mediante Application Control e inspección de capa 7.
* Bloquear las llamadas de voz/video de WhatsApp (manteniendo la mensajería de texto si aplica) mediante Application Control.
* Bloquear el acceso a `itla.edu.do` y todos sus subdominios mediante DNS Filter y URL filtering.
* Detectar y bloquear escaneos de puertos (Nmap, Nessus, etc.) dirigidos al servidor web desde Internet, mediante una **DoS Policy** en la interfaz WAN.
* Proteger el servidor web de la LAN de servidores mediante WAF (Web Application Firewall) integrado en FortiGate.

---

## 2. Topología y Direccionamiento

### 2.1 Diagrama de Topología

```
                        [ INTERNET ]
                             │
                        192.168.1.2 (ISP Gateway)
                             │
                    ┌────────┴────────┐
                    │   FortiGate     │
                    │  port1: WAN     │ 192.168.1.10/24
                    │  port2: LAN-USR │ 20.25.37.1/25
                    │  port3: LAN-SRV │ 20.25.37.129/28
                    └────┬───────┬────┘
                         │       │
            ┌────────────┘       └────────────┐
            │ port2                     port3 │
     ┌──────┴──────────┐         ┌────────────┴────────┐
     │      SW1        │         │        SW2          │
     └───┬─────────┬───┘         └──────────┬──────────┘
         │         │                        │
  ┌──────┴──┐  ┌───┴─────┐       ┌──────────┴───────┐
  │   PC1   │  │   PC2   │       │   Servidor Web   │
  │ (DHCP)  │  │ (DHCP)  │       │   20.25.37.130   │
  │20.25.37.2  │20.25.37.3       │    (Estática)    │
  └─────────┘  └─────────┘       └──────────────────┘
   LAN Usuarios 20.25.37.0/25     LAN Servidores
                                  20.25.37.128/28

  Políticas de seguridad aplicadas:
  ┌─────────────────────────────────────────────────────┐
  │ LAN-USR → Internet : NAT + App Control (bloqueo     │
  │   redes sociales, WhatsApp calls, itla.edu.do)      │
  │ LAN-USR → LAN-SRV  : Solo HTTP (puerto 80)          │
  │ Internet → LAN-SRV : WAF en tráfico web             │
  │ Internet (port1)   : DoS Policy — bloqueo de        │
  │   escaneos de puertos (tcp_port_scan / udp_scan)    │
  └─────────────────────────────────────────────────────┘
```

### 2.2 Tabla de Interfaces y Dispositivos

**Interfaces del FortiGate:**

| Interfaz | Alias | Rol | Dirección IP | Máscara | Notas |
|---|---|---|---|---|---|
| **port1** | WAN | WAN | 192.168.1.10 | /24 | Gateway ISP: 192.168.1.2 |
| **port2** | LAN-USUARIOS | LAN | 20.25.37.1 | /25 | Gateway de la LAN de usuarios |
| **port3** | LAN-SERVIDORES | LAN | 20.25.37.129 | /28 | Gateway de la LAN de servidores |

**Dispositivos en la red:**

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway | Método | Rol |
|---|---|---|---|---|---|---|
| **FortiGate** | port1 | 192.168.1.10 | /24 | 192.168.1.2 | Estática | Firewall — WAN |
| **FortiGate** | port2 | 20.25.37.1 | /25 | — | Estática | Gateway LAN Usuarios |
| **FortiGate** | port3 | 20.25.37.129 | /28 | — | Estática | Gateway LAN Servidores |
| **SW1** | — | — | — | — | — | Conmutación LAN Usuarios (conectado a port2) |
| **SW2** | — | — | — | — | — | Conmutación LAN Servidores (conectado a port3) |
| **PC1** | eth0 | 20.25.37.2 | /25 | 20.25.37.1 | **DHCP** | Cliente de usuario 1 |
| **PC2** | eth0 | 20.25.37.3 | /25 | 20.25.37.1 | **DHCP** | Cliente de usuario 2 |
| **Servidor Web** | eth0 | 20.25.37.130 | /28 | 20.25.37.129 | **Estática** | Servidor HTTP protegido con WAF |

> PC1 y PC2 reciben su IP dinámicamente del servidor DHCP configurado en port2. El rango disponible es `20.25.37.2 – 20.25.37.126` — las IPs `.2` y `.3` son los primeros leases asignados pero pueden variar. El Servidor Web tiene IP estática fija `20.25.37.130` para que la política WAF y la ruta siempre apunten a él correctamente.

---

## 3. Configuraciones por la GUI

> Toda la configuración de este laboratorio se realiza mediante la interfaz gráfica (GUI) de FortiGate, **excepto el acceso inicial** que requiere una configuración mínima por CLI para habilitar el acceso a la GUI. En cada sección se indica la ruta exacta de navegación y los parámetros configurados. Las capturas de evidencia se encuentran en la [sección 4](#4-capturas-de-pantalla).

---

### 3.0 Acceso Inicial — Configuración por CLI

Antes de poder usar la GUI, es necesario asignar una IP a la interfaz WAN desde la consola del FortiGate y habilitar el acceso administrativo en ella. Esto se realiza conectándose al puerto de consola (cable serial) o directamente en el terminal de PNETLab/EVE-NG.

```bash
config system interface
    edit "port1"
        set mode static
        set ip 192.168.1.10 255.255.255.0
        set allowaccess http https ssh ping
        set role wan
    next
end
```

Una vez aplicado, abrir un navegador desde el host físico y acceder a:

```
https://192.168.1.10
```

> **Nota importante:** Habilitar acceso administrativo (`https`, `ssh`) en la interfaz WAN **no es una práctica recomendada en producción** — expone la GUI de gestión directamente a Internet. En un entorno real, la administración se realiza desde una interfaz de gestión dedicada (OOB) o desde una LAN interna de confianza. En este laboratorio se hace así únicamente por conveniencia de acceso desde el host físico de PNETLab.

> Credenciales por defecto de FortiGate:
> - **Usuario:** `admin`
> - **Contraseña:** *(vacía — presionar Enter)*

> FortiGate pedirá cambiar la contraseña en el primer inicio de sesión. Definir una contraseña segura antes de continuar.

> Ver evidencia: [00_cli_acceso_inicial.png](#00_cli_acceso_inicialpng)

---

### 3.1 Configuración de Interfaces

**Ruta:** `Network → Interfaces`

Editar cada interfaz haciendo doble clic sobre ella:

**port1 — WAN:**

| Campo | Valor |
|---|---|
| Alias | `WAN` |
| Role | `WAN` |
| Addressing mode | `Manual` |
| IP/Netmask | `192.168.1.10 / 255.255.255.0` |
| Administrative access | `HTTPS, SSH, Ping` |

**port2 — LAN Usuarios:**

| Campo | Valor |
|---|---|
| Alias | `LAN-USUARIOS` |
| Role | `LAN` |
| Addressing mode | `Manual` |
| IP/Netmask | `20.25.37.1 / 255.255.255.128` |
| Administrative access | `HTTP, SSH, Ping` |

**port3 — LAN Servidores:**

| Campo | Valor |
|---|---|
| Alias | `LAN-SERVIDORES` |
| Role | `LAN` |
| Addressing mode | `Manual` |
| IP/Netmask | `20.25.37.129 / 255.255.255.240` |
| Administrative access | `Ping` |

> Ver evidencia: [01_interfaces_configuradas.png](#01_interfaces_configuradaspng)

---

### 3.2 Servidor DHCP en LAN de Usuarios

**Ruta:** `Network → Interfaces → port2 → Edit → DHCP Server → Create New`

| Campo | Valor |
|---|---|
| Status | `Enable` |
| Address Range | `20.25.37.2 – 20.25.37.126` |
| Netmask | `255.255.255.128` |
| Default Gateway | `20.25.37.1` |
| DNS Server | `8.8.8.8` / `8.8.4.4` |
| Lease Time | `1 day` |

> Ver evidencia: [02_dhcp_lan_usuarios.png](#02_dhcp_lan_usuariospng)

---

### 3.3 Ruta por Defecto

**Ruta:** `Network → Static Routes → Create New`

| Campo | Valor |
|---|---|
| Destination | `0.0.0.0 / 0.0.0.0` |
| Gateway | `192.168.1.2` |
| Interface | `port1` |
| Distance | `10` |
| Status | `Enable` |

> Ver evidencia: [03_ruta_default.png](#03_ruta_defaultpng)

---

### 3.4 NAT — Acceso a Internet

El NAT en FortiGate se habilita directamente en la política de firewall que permite el acceso a Internet desde la LAN de usuarios.

**Ruta:** `Policy & Objects → Firewall Policy → Create New`

| Campo | Valor |
|---|---|
| Name | `LAN-USR-to-Internet` |
| Incoming Interface | `port2 (LAN-USUARIOS)` |
| Outgoing Interface | `port1 (WAN)` |
| Source | `all` |
| Destination | `all` |
| Service | `ALL` |
| Action | `ACCEPT` |
| NAT | ✅ Enable — `Use Outgoing Interface Address` |
| Security Profiles | *(se asignan en secciones siguientes)* |

> Ver evidencia: [04_politica_nat_internet.png](#04_politica_nat_internetpng)

---

### 3.5 Política HTTP — LAN Usuarios a LAN Servidores

Esta política permite **únicamente tráfico HTTP (puerto 80)** desde la LAN de usuarios hacia la LAN de servidores. Todo otro tráfico entre ambos segmentos es denegado implícitamente.

**Ruta:** `Policy & Objects → Firewall Policy → Create New`

| Campo | Valor |
|---|---|
| Name | `LAN-USR-to-SRV-HTTP-only` |
| Incoming Interface | `port2 (LAN-USUARIOS)` |
| Outgoing Interface | `port3 (LAN-SERVIDORES)` |
| Source | `all` |
| Destination | `all` |
| Service | `HTTP` |
| Action | `ACCEPT` |
| NAT | ❌ Disabled |
| Log Allowed Traffic | `All Sessions` |

> **Importante:** Esta política debe quedar **por encima** de cualquier política que permita ALL entre estos segmentos. El orden de las políticas en FortiGate es top-down — la primera que hace match se aplica.

> Ver evidencia: [05_politica_http_solo.png](#05_politica_http_solopng)

---

### 3.6 Bloqueo de Redes Sociales

El bloqueo de redes sociales se implementa mediante un **Application Control Profile** que bloquea la categoría "Social Media".

**Paso 1 — Crear el Application Control Profile:**

**Ruta:** `Security Profiles → Application Control → Create New`

| Campo | Valor |
|---|---|
| Name | `APP-CTRL-USUARIOS` |
| Categories → Social Media | `Block` |
| Unknown Applications | `Monitor` |

**Paso 2 — Aplicar el perfil a la política de Internet:**

Editar la política `LAN-USR-to-Internet` → Security Profiles:

| Campo | Valor |
|---|---|
| Application Control | ✅ `APP-CTRL-USUARIOS` |

> El Application Control de FortiGate identifica aplicaciones de redes sociales (Facebook, Instagram, TikTok, Twitter/X, etc.) por su firma de tráfico, independientemente del puerto o protocolo que usen.

> Ver evidencia: [06_app_control_social_media.png](#06_app_control_social_mediapng)

---

### 3.7 Bloqueo de Llamadas por WhatsApp

Las llamadas de voz y video de WhatsApp se bloquean con granularidad dentro del mismo Application Control Profile, manteniendo disponible la mensajería si se desea.

**Ruta:** `Security Profiles → Application Control → APP-CTRL-USUARIOS → Edit`

En la sección **Application Overrides → Add Signatures:**

| Campo | Valor |
|---|---|
| Buscar | `WhatsApp_VoIP.Call` |
| Action | `Block` |

> FortiGate diferencia las llamadas de WhatsApp (`WhatsApp_VoIP.Call`) de la mensajería de texto (`WhatsApp`) — esto permite bloquear solo el tráfico de voz/video sin afectar los mensajes de texto.

> Ver evidencia: [07_whatsapp_calls_block.png](#07_whatsapp_calls_blockpng)

---

### 3.8 Bloqueo de itla.edu.do

El bloqueo del dominio `itla.edu.do` y todos sus subdominios se implementa mediante **DNS Filter** y **Web Filter** para cobertura completa.

**Paso 1 — DNS Filter:**

**Ruta:** `Security Profiles → DNS Filter → Create New`

| Campo | Valor |
|---|---|
| Name | `DNS-FILTER-USUARIOS` |
| Domain Filter → Add Entry | |
| Domain | `itla.edu.do` |
| Type | `Wildcard` |
| Action | `Block` |

> El wildcard `itla.edu.do` cubre automáticamente `www.itla.edu.do`, `moodle.itla.edu.do`, `correo.itla.edu.do` y cualquier otro subdominio.

**Paso 2 — Web Filter (URL Filter):**

**Ruta:** `Security Profiles → Web Filter → Create New`

| Campo | Valor |
|---|---|
| Name | `WEB-FILTER-USUARIOS` |
| URL Filter → Create New | |
| URL | `*.itla.edu.do` |
| Type | `Wildcard` |
| Action | `Block` |

**Paso 3 — Aplicar ambos perfiles a la política de Internet:**

Editar `LAN-USR-to-Internet` → Security Profiles:

| Campo | Valor |
|---|---|
| DNS Filter | ✅ `DNS-FILTER-USUARIOS` |
| Web Filter | ✅ `WEB-FILTER-USUARIOS` |

> Ver evidencia: [08_dns_web_filter_itla.png](#08_dns_web_filter_itlapng)

---

### 3.9 Detección y Bloqueo de Escaners de Red

La detección y bloqueo de escaneos de puertos (Nmap, Nessus, y similares) **no se implementa con IPS Signatures ni con Application Control** — no existen firmas de IPS con nombres genéricos como `Network.Scan.*` o `Port.Scan` en FortiOS, y la firma de aplicación `Portmap` solo cubre tráfico al protocolo *portmapper/rpcbind*, no un escaneo de puertos general con Nmap.

El mecanismo correcto que provee FortiOS para esto es una **DoS Policy** (`Policy & Objects → DoS Policy`), que detecta el comportamiento del tráfico — volumen de conexiones o paquetes por segundo hacia múltiples puertos — en vez de depender de una firma fija.

**Consideración de diseño — ¿en qué interfaz aplicarla?**

Una DoS Policy solo inspecciona el tráfico que **entra** por la interfaz configurada (`Incoming Interface`). El objetivo de este laboratorio es proteger el servidor web contra reconocimiento hecho por un atacante externo, así que la policy debe aplicarse en **port1 (WAN)** — el tráfico de un escaneo interno entre PCs de la LAN de usuarios queda fuera del alcance de este objetivo y no se cubre aquí.

> Nota técnica: en FortiGate solo puede existir **una DoS Policy activa por interfaz**. Si más adelante se necesita cubrir escaneos originados dentro de la LAN de usuarios, se debe crear una segunda policy con `Incoming Interface: port2`, ya que no se pueden apilar varias DoS Policies sobre la misma interfaz.

**Configuración:**

**Ruta:** `Policy & Objects → DoS Policy → Create New`

| Campo | Valor |
|---|---|
| Name | `DOS-ANTI-SCAN-WAN` |
| Incoming Interface | `port1 (WAN)` |
| Source Address | `all` |
| Destination Address | `20.25.37.130` (IP del Servidor Web) |
| Service | `ALL` |

**Anomalías (L4 Anomalies) a configurar dentro de la misma policy:**

| Anomaly | Logging | Action | Threshold (lab) | Threshold (default) |
|---|---|---|---|---|
| `tcp_port_scan` | ✅ Enable | **`Block`** | `5` | 1000 |
| `udp_scan` | ✅ Enable | **`Block`** | `5` | 2000 |

> **Importante:** al crear la anomalía, habilitar el toggle de **Logging** no es suficiente — hay que hacer clic explícitamente en el botón **`Block`** dentro de la columna Action (por defecto queda en `Disable`, que solo registra sin bloquear).

> **Sobre el threshold:** los valores por defecto de Fortinet (1000–2000 paquetes/seg) están pensados para tráfico de producción a gran escala, no para verse en una demo corta de laboratorio. Bajarlos a `5` hace que un escaneo de Nmap dispare el bloqueo casi de inmediato.

**Verificación práctica:**

Desde el host físico (fuera del FortiGate, simulando Internet), correr:

```bash
nmap -sS 192.168.1.10
```

o directamente contra la IP pública/NAT que apunte al servidor web.

**Dónde revisar el resultado:**

Los eventos de DoS **no aparecen en `Log & Report → Forward Traffic`** (error corregido respecto a una versión anterior de este documento). Se revisan en:

```
Log & Report → Security Events → Anomaly
```

Ahí se debe ver el origen del escaneo, la anomalía disparada (`tcp_port_scan` / `udp_scan`), la acción `Blocked` y el conteo de paquetes que superó el threshold.

> Ver evidencia: [09_dos_anti_scan.png](#09_dos_anti_scanpng)

---

### 3.10 WAF — Protección del Servidor Web

El WAF (Web Application Firewall) de FortiGate se aplica al tráfico HTTP entrante hacia el servidor web en la LAN de servidores, protegiendo contra SQL Injection, XSS, Path Traversal y otros ataques de capa de aplicación.

**Paso 1 — Crear el WAF Profile:**

**Ruta:** `Security Profiles → Web Application Firewall → Create New`

| Campo | Valor |
|---|---|
| Name | `WAF-SERVIDOR-WEB` |
| Signature groups → Main | ✅ Enable — Action: `Block` |
| Constraint | ✅ Enable — Action: `Block` |
| Method | ✅ Enable — Allow: `GET, POST, HEAD` |

Dentro de **Signature groups → Main**, habilitar:

| Grupo de firmas | Acción |
|---|---|
| `SQL Injection` | `Block` |
| `XSS Attack` | `Block` |
| `Generic Attacks` | `Block` |
| `Known Exploits` | `Block` |

**Paso 2 — Crear política para tráfico hacia el servidor web:**

**Ruta:** `Policy & Objects → Firewall Policy → Create New`

| Campo | Valor |
|---|---|
| Name | `Internet-to-WebServer-WAF` |
| Incoming Interface | `port1 (WAN)` |
| Outgoing Interface | `port3 (LAN-SERVIDORES)` |
| Source | `all` |
| Destination | `20.25.37.130` (IP del servidor web) |
| Service | `HTTP, HTTPS` |
| Action | `ACCEPT` |
| Web Application Firewall | ✅ `WAF-SERVIDOR-WEB` |
| Log Allowed Traffic | `All Sessions` |

> Ver evidencia: [10_waf_servidor_web.png](#10_waf_servidor_webpng)

---

## 4. Capturas de Pantalla

Las siguientes capturas de pantalla documentan cada punto de configuración de la GUI y están almacenadas en la carpeta [`screenshots/`](screenshots/).

| # | Archivo | Descripción |
|---|---|---|
| 00 | [`00_cli_acceso_inicial.png`](screenshots/00_cli_acceso_inicial.png) | Terminal CLI del FortiGate mostrando la configuración de port1 con IP `192.168.1.10/24` y `allowaccess http https ssh ping`, seguido de la pantalla de login de la GUI en el navegador. |
| 01 | [`01_interfaces_configuradas.png`](screenshots/01_interfaces_configuradas.png) | Vista de `Network → Interfaces` mostrando port1 (WAN: 192.168.1.10/24), port2 (LAN-USUARIOS: 20.25.37.1/25) y port3 (LAN-SERVIDORES: 20.25.37.129/28) con sus roles y estados. |
| 02 | [`02_dhcp_lan_usuarios.png`](screenshots/02_dhcp_lan_usuarios.png) | Vista del servidor DHCP configurado en port2 mostrando el rango `20.25.37.2 – 20.25.37.126`, el gateway `20.25.37.1` y los servidores DNS. |
| 03 | [`03_ruta_default.png`](screenshots/03_ruta_default.png) | Vista de `Network → Static Routes` mostrando la ruta `0.0.0.0/0` apuntando al gateway `192.168.1.2` por port1. |
| 04 | [`04_politica_nat_internet.png`](screenshots/04_politica_nat_internet.png) | Política `LAN-USR-to-Internet` mostrando src: port2, dst: port1, acción ACCEPT con NAT habilitado usando la IP de la interfaz saliente. |
| 05 | [`05_politica_http_solo.png`](screenshots/05_politica_http_solo.png) | Política `LAN-USR-to-SRV-HTTP-only` mostrando src: port2, dst: port3, servicio: HTTP únicamente, acción ACCEPT — posicionada sobre cualquier política ALL entre esos segmentos. |
| 06 | [`06_app_control_social_media.png`](screenshots/06_app_control_social_media.png) | Perfil `APP-CTRL-USUARIOS` en `Security Profiles → Application Control` mostrando la categoría "Social Media" con acción Block. |
| 07 | [`07_whatsapp_calls_block.png`](screenshots/07_whatsapp_calls_block.png) | Vista de Application Overrides en el perfil `APP-CTRL-USUARIOS` mostrando `WhatsApp_VoIP.Call` con acción Block. |
| 08 | [`08_dns_web_filter_itla.png`](screenshots/08_dns_web_filter_itla.png) | Vista del DNS Filter mostrando la entrada wildcard `itla.edu.do` con acción Block, y el Web Filter con `*.itla.edu.do` bloqueado. |
| 09 | [`09_dos_anti_scan.png`](screenshots/09_dos_anti_scan.png) | Policy `DOS-ANTI-SCAN-WAN` en `Policy & Objects → DoS Policy` mostrando Incoming Interface `port1`, destino `20.25.37.130`, y las anomalías `tcp_port_scan` y `udp_scan` en acción Block con threshold ajustado a 5. |
| 10 | [`10_waf_servidor_web.png`](screenshots/10_waf_servidor_web.png) | Perfil WAF `WAF-SERVIDOR-WEB` mostrando los grupos de firmas SQL Injection, XSS y Generic Attacks habilitados con acción Block. |
| 11 | [`11_politica_waf_aplicada.png`](screenshots/11_politica_waf_aplicada.png) | Política `Internet-to-WebServer-WAF` mostrando el perfil WAF asignado en Security Profiles. |
| 12 | [`12_tabla_politicas_orden.png`](screenshots/12_tabla_politicas_orden.png) | Vista completa de `Policy & Objects → Firewall Policy` mostrando todas las políticas en el orden correcto de aplicación. |
| 13 | [`13_dhcp_leases.png`](screenshots/13_dhcp_leases.png) | Vista de leases DHCP activos en port2 mostrando al menos un cliente con IP asignada del rango, confirmando que el DHCP funciona. |
| 14 | [`14_ping_internet_ok.png`](screenshots/14_ping_internet_ok.png) | Ping exitoso desde un cliente de la LAN de usuarios hacia `8.8.8.8` — confirma NAT y ruta por defecto funcionando. |
| 15 | [`15_http_a_servidor_ok.png`](screenshots/15_http_a_servidor_ok.png) | Acceso HTTP exitoso desde la LAN de usuarios al servidor web (`20.25.37.130`) — confirma la política HTTP-only. |
| 16 | [`16_bloqueo_rrss.png`](screenshots/16_bloqueo_rrss.png) | Intento fallido de acceso a una red social (ej. facebook.com) desde la LAN de usuarios — página de bloqueo de FortiGate. |
| 17 | [`17_bloqueo_itla.png`](screenshots/17_bloqueo_itla.png) | Intento fallido de acceso a `itla.edu.do` y un subdominio (ej. `moodle.itla.edu.do`) — página de bloqueo de FortiGate. |
| 18 | [`18_log_forward_traffic.png`](screenshots/18_log_forward_traffic.png) | Vista de `Log & Report → Forward Traffic` mostrando entradas de tráfico aceptado (HTTP a servidor) y bloqueado (red social) con IPs y políticas aplicadas. |
| 19 | [`19_log_anomaly_scan.png`](screenshots/19_log_anomaly_scan.png) | Vista de `Log & Report → Security Events → Anomaly` mostrando el evento `tcp_port_scan` bloqueado tras correr Nmap contra el servidor web desde el host físico. |
| 20 | [`20_waf_sqli_bloqueado.png`](screenshots/20_waf_sqli_bloqueado.png) | Intento de SQL Injection (`' OR '1'='1' --`) contra el formulario de login del servidor web desde WAN — página de bloqueo de FortiGate mostrando la firma de ataque detectada (SQL Injection) por el perfil `WAF-SERVIDOR-WEB`. |
| 21 | [`21_log_waf_sqli.png`](screenshots/21_log_waf_sqli.png) | Vista de `Log & Report → Security Events` (filtro Application/WAF) mostrando el evento bloqueado con la firma SQL Injection, IP origen y la acción `Blocked`. |

---

## 5. Video Demostrativo

🎥 **[Ver demostración en YouTube](https://youtu.be/i2C1Tw8JZUg)**

**Duración:** 6:31

**Contenido del video:**

* ✅ Topología en FortiGate con nombre completo `Jordy Rosario — 20250737` visible.
* ✅ Reloj del sistema operativo visible evidenciando fecha y hora actual.
* ✅ Rostro y voz del autor realizando la explicación técnica.
* ✅ Demostración de las interfaces configuradas (`Network → Interfaces`).
* ✅ Verificación del DHCP con un cliente recibiendo IP del pool.
* ✅ Ping exitoso desde la LAN de usuarios a Internet (NAT funcionando).
* ✅ Acceso HTTP al servidor web desde la LAN de usuarios.
* ✅ Bloqueo demostrado: red social inaccesible desde la LAN de usuarios.
* ✅ Bloqueo demostrado: `itla.edu.do` inaccesible con mensaje de bloqueo.
* ✅ Escaneo de puertos (`nmap -sS`) contra el servidor web desde el host físico, mostrando el bloqueo en `Log & Report → Security Events → Anomaly`.
* ✅ Vista de `Log & Report → Forward Traffic` con los logs de tráfico aceptado y bloqueado.

---

## 6. Referencias

* Fortinet. (2024). *FortiGate Administration Guide 7.x — Firewall Policies*.
* Fortinet. (2024). *FortiGate Administration Guide 7.x — Application Control*.
* Fortinet. (2024). *FortiGate Administration Guide 7.x — Web Application Firewall*.
* Fortinet. (2024). *FortiGate Administration Guide 7.x — DoS Policy*.
* Fortinet. (2024). *FortiGate Administration Guide 7.x — DNS Filter*.
* Fortinet Community. *Technical Tip: How to block Port Scan or Port Scanning application*. community.fortinet.com.
* OWASP. (2024). *OWASP Top 10 Web Application Security Risks*. owasp.org.
