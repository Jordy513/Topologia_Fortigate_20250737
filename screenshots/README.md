# Capturas de pantalla — FortiGate Topología de Seguridad

Capturas del laboratorio en orden de demostración.

| # | Archivo | Descripción |
|---|---|---|
| 00 | [`00_cli_acceso_inicial.png`](/screenshots/00_cli_acceso_inicial.png) | Terminal CLI del FortiGate mostrando la configuración de port1 con IP `192.168.1.10/24` y `allowaccess http https ssh ping`, seguido de la pantalla de login de la GUI en el navegador. |
| 01 | [`01_interfaces_configuradas.png`](/screenshots/01_interfaces_configuradas.png) | Vista de `Network → Interfaces` mostrando port1 (WAN: 192.168.1.10/24), port2 (LAN-USUARIOS: 20.25.37.1/25) y port3 (LAN-SERVIDORES: 20.25.37.129/28) con sus roles y estados. |
| 02 | [`02_dhcp_lan_usuarios.png`](/screenshots/02_dhcp_lan_usuarios.png) | Vista del servidor DHCP configurado en port2 mostrando el rango `20.25.37.2 – 20.25.37.126`, el gateway `20.25.37.1` y los servidores DNS. |
| 03 | [`03_ruta_default.png`](/screenshots/03_ruta_default.png) | Vista de `Network → Static Routes` mostrando la ruta `0.0.0.0/0` apuntando al gateway `192.168.1.2` por port1. |
| 04 | [`04_politica_nat_internet.png`](/screenshots/04_politica_nat_internet.png) | Política `LAN-USR-to-Internet` mostrando src: port2, dst: port1, acción ACCEPT con NAT habilitado usando la IP de la interfaz saliente. |
| 05 | [`05_politica_http_solo.png`](/screenshots/05_politica_http_solo.png) | Política `LAN-USR-to-SRV-HTTP-only` mostrando src: port2, dst: port3, servicio: HTTP únicamente, acción ACCEPT — posicionada sobre cualquier política ALL entre esos segmentos. |
| 06 | [`06_app_control_social_media.png`](/screenshots/06_app_control_social_media.png) | Perfil `APP-CTRL-USUARIOS` en `Security Profiles → Application Control` mostrando la categoría "Social Media" con acción Block. |
| 07 | [`07_whatsapp_calls_block.png`](/screenshots/07_whatsapp_calls_block.png) | Vista de Application Overrides en el perfil `APP-CTRL-USUARIOS` mostrando `WhatsApp_VoIP.Call` con acción Block. |
| 08 | [`08_dns_web_filter_itla.png`](/screenshots/08_dns_web_filter_itla.png) | Vista del DNS Filter mostrando la entrada wildcard `itla.edu.do` con acción Block, y el Web Filter con `*.itla.edu.do` bloqueado. |
| 09 | [`09_dos_anti_scan.png`](/screenshots/09_dos_anti_scan.png) | Policy `DOS-ANTI-SCAN-WAN` en `Policy & Objects → DoS Policy` mostrando Incoming Interface `port1`, destino `20.25.37.130`, y las anomalías `tcp_port_scan` y `udp_scan` en acción Block con threshold ajustado a 5. |
| 10 | [`10_waf_servidor_web.png`](/screenshots/10_waf_servidor_web.png) | Perfil WAF `WAF-SERVIDOR-WEB` mostrando los grupos de firmas SQL Injection, XSS y Generic Attacks habilitados con acción Block. |
| 11 | [`11_politica_waf_aplicada.png`](/screenshots/11_politica_waf_aplicada.png) | Política `Internet-to-WebServer-WAF` mostrando el perfil WAF asignado en Security Profiles. |
| 12 | [`12_tabla_politicas_orden.png`](/screenshots/12_tabla_politicas_orden.png) | Vista completa de `Policy & Objects → Firewall Policy` mostrando todas las políticas en el orden correcto de aplicación. |
| 13 | [`13_dhcp_leases.png`](/screenshots/13_dhcp_leases.png) | Vista de leases DHCP activos en port2 mostrando al menos un cliente con IP asignada del rango, confirmando que el DHCP funciona. |
| 14 | [`14_ping_internet_ok.png`](/screenshots/14_ping_internet_ok.png) | Ping exitoso desde un cliente de la LAN de usuarios hacia `8.8.8.8` — confirma NAT y ruta por defecto funcionando. |
| 15 | [`15_http_a_servidor_ok.png`](/screenshots/15_http_a_servidor_ok.png) | Acceso HTTP exitoso desde la LAN de usuarios al servidor web (`20.25.37.130`) — confirma la política HTTP-only. |
| 16 | [`16_bloqueo_rrss.png`](/screenshots/16_bloqueo_rrss.png) | Intento fallido de acceso a una red social (ej. facebook.com) desde la LAN de usuarios — página de bloqueo de FortiGate. |
| 17 | [`17_bloqueo_itla.png`](/screenshots/17_bloqueo_itla.png) | Intento fallido de acceso a `itla.edu.do` y un subdominio (ej. `moodle.itla.edu.do`) — página de bloqueo de FortiGate. |
| 18 | [`18_log_forward_traffic.png`](/screenshots/18_log_forward_traffic.png) | Vista de `Log & Report → Forward Traffic` mostrando entradas de tráfico aceptado (HTTP a servidor) y bloqueado (red social) con IPs y políticas aplicadas. |
| 19 | [`19_log_anomaly_scan.png`](/screenshots/19_log_anomaly_scan.png) | Vista de `Log & Report → Security Events → Anomaly` mostrando el evento `tcp_port_scan` bloqueado tras correr Nmap contra el servidor web desde el host físico. |
| 20 | [`20_waf_sqli_bloqueado.png`](/screenshots/20_waf_sqli_bloqueado.png) | Intento de SQL Injection (`' OR '1'='1' --`) contra el formulario de login del servidor web desde WAN — página de bloqueo de FortiGate mostrando la firma de ataque detectada (SQL Injection) por el perfil `WAF-SERVIDOR-WEB`. |
| 21 | [`21_log_waf_sqli.png`](/screenshots/21_log_waf_sqli.png) | Vista de `Log & Report → Security Events` (filtro Application/WAF) mostrando el evento bloqueado con la firma SQL Injection, IP origen y la acción `Blocked`. |

