# Módulo: Planificación y Administración de Redes (0370)
## Agencia Espanito — Infraestructura de Red

**Proyecto Intermodular · 1º ASIR · Yassin Mansouri**

---

## 1. Análisis de Necesidades de Red

### Descripción del entorno

La agencia Espanito dispone de ocho dependencias con los siguientes equipos:

| Dependencia | Usuarios | PCs | Laptops | IP Phones | Impresoras | Tablets | Móviles | TV | AP | Total |
|---|---|---|---|---|---|---|---|---|---|---|
| Dirección | 1 | 1 | 1 | 1 | 1 | — | 1 | — | 1 | 6 |
| Asesoramiento | 4 | 4 | — | 4 | 2 | — | 4 | — | 1 | 15 |
| Diseño / Redes Sociales | 2 | 2 | — | 2 | — | 2 | 2 | — | 1 | 9 |
| Secretaría | 1 | 1 | — | 1 | — | — | 1 | 1 | — | 4 |
| Sala de Red (IT) | 1 | 1 | — | — | — | — | 1 | — | 1 | 3 + servidor |
| Aula 1 | 6 | 6 | — | — | — | — | — | — | — | 6 |
| Aula 2 | 6 | 6 | — | — | — | — | — | — | — | 6 |
| Aula 3 | 6 | 6 | — | — | — | — | — | — | — | 6 |
| **Total** | **27** | **27** | **1** | **8** | **3** | **2** | **9** | **1** | **4** | **~55** |

> Los teléfonos IP de las aulas (Aula1, Aula2, Aula3) se conectan directamente al switch de cada aula. En Packet Tracer se simulan como dispositivos separados en VLAN 50.

### Necesidades identificadas

- Acceso a Internet para todos los equipos
- Telefonía interna VoIP gestionada por CME en el Router 2811
- Segmentación por VLANs para aislar zonas funcionales
- Servicio DHCP centralizado en el servidor para todas las VLANs
- Resolución DNS interna (`espanito.local`) y reenvío DNS externo
- Acceso a `espanito_db` desde asesoramiento, dirección y secretaría
- WiFi en dirección, asesoramiento, diseño y sala de red para dispositivos móviles y tablets

### Requisitos de seguridad

- Las aulas (VLANs 200, 210, 220) no tienen acceso a los servidores (VLAN 10)
- Las aulas no tienen acceso a las VLANs administrativas (100–150)
- El tráfico VoIP en VLAN 50 está aislado del resto del tráfico de datos
- Solo el administrador (VLAN 150) tiene acceso completo a todos los segmentos

---

## 2. Diseño de la Topología de Red

### Topología: estrella jerárquica (dos niveles)

```
           ┌─────────────────────────────────┐
           │         INTERNET / ISP          │
           └────────────────┬────────────────┘
                            │ Fa0/1 (WAN)
                ┌───────────┴───────────┐
                │   ROUTER CISCO 2811   │  ← CME (VoIP) + Gateway Internet
                │     172.20.0.2/30     │
                └───────────┬───────────┘
                            │ Fa0/0
                            │ 172.20.0.1/30
                ┌───────────┴───────────┐
                │  L3 SWITCH CISCO 3560 │  ← Inter-VLAN Routing
                │   (ip routing ON)     │     ip helper-address → Servidor
                └────────┬─────────────┘
                         │
      ┌──────────────────┼────────────────────┐
      │              Trunk 802.1Q              │
   Fa0/1            Fa0/2      Fa0/3         Fa0/4
SW-Admin          SW-Consulta  SW-Diseño   SW-Secretaria
(2960)              (2960)      (2960)       (2960)
VLAN 50,100       VLAN 50,120  VLAN 50,130  VLAN 50,140

   Fa0/5            Fa0/6      Fa0/7
 SW-Aula1         SW-Aula2   SW-Aula3
  (2960)           (2960)     (2960)
 VLAN 200         VLAN 210   VLAN 220

   Fa0/20           Fa0/21
  Servidor         PC Sala Red
 172.20.10.10       VLAN 150
 (VLAN 10 — acceso)
```

### Justificación del diseño

El L3 Switch 3560 centraliza el enrutamiento entre VLANs mediante SVIs (Switch Virtual Interfaces), actuando como gateway de cada subred. El Router 2811 gestiona la conectividad WAN y aloja el servicio CME (Call Manager Express) para los teléfonos IP. El servidor en VLAN 10 centraliza DHCP y DNS para toda la red gracias al mecanismo `ip helper-address` configurado en cada SVI del switch L3.

---

## 3. Plan de Direccionamiento IP

### Rango base: `172.20.0.0/16`

| VLAN | Nombre | Red | Máscara | Gateway (SVI) | DHCP pool | Broadcast |
|---|---|---|---|---|---|---|
| — | Enlace Router-L3 | 172.20.0.0 | /30 | — | No (ptp) | 172.20.0.3 |
| 10 | Servidores | 172.20.10.0 | /24 | 172.20.10.1 | No (IPs fijas) | 172.20.10.255 |
| 50 | VoIP | 172.20.50.0 | /24 | 172.20.50.1 | 172.20.50.100–119 | 172.20.50.255 |
| 100 | Dirección | 172.20.100.0 | /24 | 172.20.100.1 | 172.20.100.10–30 | 172.20.100.255 |
| 120 | Asesoramiento | 172.20.120.0 | /24 | 172.20.120.1 | 172.20.120.10–30 | 172.20.120.255 |
| 130 | Diseño/SM | 172.20.130.0 | /24 | 172.20.130.1 | 172.20.130.10–20 | 172.20.130.255 |
| 140 | Secretaría | 172.20.140.0 | /24 | 172.20.140.1 | 172.20.140.10–15 | 172.20.140.255 |
| 150 | Sala de Red | 172.20.150.0 | /24 | 172.20.150.1 | 172.20.150.10–15 | 172.20.150.255 |
| 200 | Aula 1 | 172.20.200.0 | /24 | 172.20.200.1 | 172.20.200.10–15 | 172.20.200.255 |
| 210 | Aula 2 | 172.20.210.0 | /24 | 172.20.210.1 | 172.20.210.10–15 | 172.20.210.255 |
| 220 | Aula 3 | 172.20.220.0 | /24 | 172.20.220.1 | 172.20.220.10–15 | 172.20.220.255 |

### IPs fijas

| Dispositivo | VLAN | Dirección IP | Función |
|---|---|---|---|
| Router 2811 (Fa0/0) | Enlace | 172.20.0.2/30 | Gateway WAN + CME |
| L3 Switch 3560 (Gi0/1) | Enlace | 172.20.0.1/30 | Uplink al router |
| Servidor Espanito | 10 | 172.20.10.10/24 | DHCP + DNS centralizado |

### Extensiones VoIP (VLAN 50 — DHCP con TFTP=172.20.0.2)

| Extensión | Ubicación | IP esperada |
|---|---|---|
| 100 | Dirección | 172.20.50.100 |
| 101–104 | Asesores 1–4 | 172.20.50.101–104 |
| 105–106 | Diseño 1–2 | 172.20.50.105–106 |
| 107 | Secretaría | 172.20.50.107 |

---

## 4. Dispositivos de Red

### Router Cisco 2811

**Función:** Conectividad WAN + CME (Call Manager Express) para telefonía VoIP interna.

- `Fa0/0`: Enlace LAN hacia L3 Switch — 172.20.0.2/30
- `Fa0/1`: Enlace WAN al ISP (DHCP o IP pública estática)
- Protocolo de enrutamiento: ruta estática por defecto hacia ISP + ruta estática `172.20.0.0/16` hacia L3 Switch
- CME: gestiona las extensiones SIP de todos los IP Phones de la agencia (ephone-dn 100–107)

---

### L3 Switch Cisco Catalyst 3560

**Función:** Núcleo de la red. Enrutamiento inter-VLAN mediante SVIs. Relay DHCP mediante `ip helper-address` en cada SVI.

- 10 SVIs configuradas (VLAN 10, 50, 100, 120, 130, 140, 150, 200, 210, 220)
- Puerto `Gi0/1` como puerto enrutado (no switchport) hacia el router
- 7 puertos trunk hacia los switches de acceso (802.1Q)
- 2 puertos de acceso: servidor (VLAN 10) + PC sala de red (VLAN 150)
- Ruta por defecto: `0.0.0.0/0 → 172.20.0.2`

---

### Access Switches Cisco 2960 (× 7)

| Switch | Dependencia | VLANs | Dispositivos conectados |
|---|---|---|---|
| SW-Admin | Dirección | 100, 50 | PC, Laptop, IP Phone, Impresora, AP |
| SW-Consulta | Asesoramiento | 120, 50 | 4 PCs, 4 IP Phones, 2 Impresoras, AP |
| SW-Diseño | Diseño/SM | 130, 50 | 2 PCs, 2 IP Phones, AP |
| SW-Secretaria | Secretaría | 140, 50 | PC, TV, IP Phone |
| SW-Aula1 | Aula 1 | 200 | 6 PCs |
| SW-Aula2 | Aula 2 | 210 | 6 PCs |
| SW-Aula3 | Aula 3 | 220 | 6 PCs |

Todos los puertos con IP Phone tienen configurado `switchport voice vlan 50` adicionalmente a la VLAN de datos correspondiente.

---

### Servidor Espanito (VLAN 10 — 172.20.10.10)

**Funciones:**
- **DHCP:** un pool por cada VLAN con `ip helper-address` redirigiendo las solicitudes desde el L3 Switch. El pool de VLAN 50 incluye TFTP Server = 172.20.0.2 para que los IP Phones descarguen configuración CME.
- **DNS:** servicio DNS activo con registros A para `espanito.local` y reenvío externo a 8.8.8.8.

---

## 5. Servicios de Red

### DHCP (centralizado en servidor 172.20.10.10)

El servidor centraliza todos los pools DHCP. El L3 Switch reenvía los broadcasts DHCP de cada VLAN al servidor mediante `ip helper-address 172.20.10.10` configurado en cada SVI.

| VLAN | Rango DHCP | TFTP (VoIP) |
|---|---|---|
| 50 (VoIP) | .100 – .119 | 172.20.0.2 ← CME Router |
| 100 (Dir.) | .10 – .30 | — |
| 120 (Ases.) | .10 – .30 | — |
| 130 (Dis.) | .10 – .20 | — |
| 140 (Secr.) | .10 – .15 | — |
| 150 (Red) | .10 – .15 | — |
| 200–220 (Aulas) | .10 – .15 | — |

### DNS (servidor 172.20.10.10)

Registros A internos configurados:
- `servidor.espanito.local` → 172.20.10.10
- `gateway.espanito.local` → 172.20.0.1
- `router-voip.espanito.local` → 172.20.0.2

Reenvío externo: 8.8.8.8 (Google DNS)

### VoIP — CME en Router 2811

El Router 2811 actúa como centralita interna (Call Manager Express). Los teléfonos IP reciben IP por DHCP (VLAN 50) y descargan su configuración automáticamente vía TFTP desde 172.20.0.2 (el propio router). Las extensiones van de 100 a 107.

### Control de acceso entre VLANs

| Origen | Destino | Política |
|---|---|---|
| VLANs 100–150 (personal) | VLAN 10 (servidores) | Permitido |
| VLANs 200–220 (aulas) | VLAN 10 (servidores) | Denegado (ACL) |
| VLANs 200–220 (aulas) | VLANs 100–150 | Denegado (ACL) |
| VLAN 50 (VoIP) | Todas | Solo tráfico SIP/RTP |

---

## 6. Coherencia con el proyecto global

La infraestructura de red diseñada sirve de base al resto de módulos del Proyecto Intermodular:

- **0372 Bases de Datos:** `espanito_db` alojada en el servidor (VLAN 10), accesible desde VLANs de personal mediante TCP/3306
- **0373 Lenguajes de Marcas:** los XML de exportación de solicitudes se almacenan en el servidor (VLAN 10)
- **0369 Sistemas Operativos:** los sistemas instalados en el servidor son administrados desde VLAN 150 (sala de red)
- **0371 Hardware:** el servidor físico, el router y el L3 switch se ubican en la sala de red (VLAN 150)
