# Módulo: Planificación y Administración de Redes (0370)
### Proyecto Intermodular · 1º ASIR · Yassin Mansouri · Agencia Espanito

---

## Resumen

Red corporativa completa para la agencia Espanito. Segmentación en 10 VLANs, telefonía VoIP con CME, DHCP centralizado en servidor y topología estrella jerárquica. Implementada y verificada en Cisco Packet Tracer.

---

## Dispositivos de red

| Dispositivo | Modelo | Función |
|---|---|---|
| Router | Cisco 2811 | Gateway WAN + CME VoIP |
| Core Switch L3 | Cisco Catalyst 3560 | Enrutamiento inter-VLAN |
| SW-Admin | Cisco Catalyst 2960 | Dirección |
| SW-Consulta | Cisco Catalyst 2960 | Asesoramiento |
| SW-Diseño | Cisco Catalyst 2960 | Diseño / Redes Sociales |
| SW-Secretaria | Cisco Catalyst 2960 | Secretaría |
| SW-Aula1 | Cisco Catalyst 2960 | Aula 1 |
| SW-Aula2 | Cisco Catalyst 2960 | Aula 2 |
| SW-Aula3 | Cisco Catalyst 2960 | Aula 3 |
| Servidor | Generic Server PT | DHCP + DNS centralizado |

---

## VLANs y direccionamiento

| VLAN | Nombre | Red | Gateway |
|---|---|---|---|
| 10 | Servidores | 172.20.10.0/24 | 172.20.10.1 |
| 50 | VoIP | 172.20.50.0/24 | 172.20.50.1 |
| 100 | Dirección | 172.20.100.0/24 | 172.20.100.1 |
| 120 | Asesoramiento | 172.20.120.0/24 | 172.20.120.1 |
| 130 | Diseño/SM | 172.20.130.0/24 | 172.20.130.1 |
| 140 | Secretaría | 172.20.140.0/24 | 172.20.140.1 |
| 150 | Sala de Red | 172.20.150.0/24 | 172.20.150.1 |
| 200 | Aula 1 | 172.20.200.0/24 | 172.20.200.1 |
| 210 | Aula 2 | 172.20.210.0/24 | 172.20.210.1 |
| 220 | Aula 3 | 172.20.220.0/24 | 172.20.220.1 |

---

## Servicios de red

- **DHCP** — centralizado en servidor 172.20.10.10, relay via `ip helper-address` en L3 Switch
- **DNS** — servidor 172.20.10.10, registros A para `espanito.local`
- **VoIP** — CME en Router 2811, extensiones 100–107, VLAN 50
- **Inter-VLAN routing** — L3 Switch 3560 con SVIs y `ip routing`
- **Acceso a Internet** — Router 2811 Fa0/1 → ISP

---

## Archivos incluidos

| Archivo | Descripción |
|---|---|
| `red_espanito.md` | Análisis completo: necesidades, topología, IPs, dispositivos, servicios |
| `diagrama_red.html` | Diagrama de topología (abrir en navegador) |
| `packet_tracer_config_espanito.md` | Configuración CLI completa para Packet Tracer |
| `README.md` | Este archivo |
