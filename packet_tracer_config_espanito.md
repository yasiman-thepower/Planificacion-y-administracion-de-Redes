# Packet Tracer — Configuración Completa Red Espanito
## Módulo: Planificación y Direccionistración de Redes (0370)
### Yassin Mansouri · 1º ASIR · Agencia Espanito

---

## 0. Resumen de topología y conexiones físicas

```
INTERNET
    │
[Router 2811]  Fa0/0 ──── Gi0/1 [L3 Switch 3560 — CORE]
172.20.0.2/30             172.20.0.1/30
                               │
              ┌────────────────┼────────────────┐
           Fa0/1           Fa0/2            Fa0/3
        SW-Direccion        SW-Consulta       SW-Diseño
        (2960)           (2960)            (2960)
              │
          Fa0/4          Fa0/5            Fa0/6            Fa0/7
       SW-Secretaria   SW-Aula1         SW-Aula2          SW-Aula3
         (2960)          (2960)           (2960)            (2960)
              │
          Fa0/20         Fa0/21           Fa0/22
          Servidor      PC-SalaRed         AP
         172.20.10.10   DHCP-VLAN150    DHCP-VLAN150
```

### Puertos del L3 Switch 3560 asignados

| Puerto L3 | Destino | Tipo | VLANs |
|---|---|---|---|
| Gi0/1 | Router 2811 Fa0/0 | Routed (no switchport) | — |
| Fa0/1 | SW-Direccion | Trunk | 50, 100 |
| Fa0/2 | SW-Consulta | Trunk | 50, 120 |
| Fa0/3 | SW-Diseño | Trunk | 50, 130 |
| Fa0/4 | SW-Secretaria | Trunk | 50, 140 |
| Fa0/5 | SW-Aula1 | Trunk | 200 |
| Fa0/6 | SW-Aula2 | Trunk | 210 |
| Fa0/7 | SW-Aula3 | Trunk | 220 |
| Fa0/20 | Servidor | Access | 10 |
| Fa0/21 | PC Sala Red | Access | 150 |
| Fa0/22 | AP Sala Red | Access | 150 |

---

## 1. ROUTER 2811 — VoIP / CME

> Conectar: Fa0/0 → Puerto Gi0/1 del L3 Switch

```
enable
configure terminal
hostname Router-Espanito-voip

! === INTERFACES ===
interface FastEthernet0/0
 description Enlace hacia L3-Switch-3560
 ip address 172.20.0.2 255.255.255.252
 no shutdown

interface FastEthernet0/1
 description WAN - Internet ISP
 ip address dhcp
 no shutdown

! === RUTAS ===
ip route 172.20.0.0 255.255.0.0 172.20.0.1
ip route 0.0.0.0 0.0.0.0 FastEthernet0/1

! === VoIP — CALL MANAGER EXPRESS (CME) ===
telephony-service
 max-ephones 15
 max-dn 15
 ip source-address 172.20.0.2 port 2000
 auto assign 1 to 9
 auto-reg-ephone

! === DIRECTORIOS TELEFÓNICOS (ephone-dn) ===
ephone-dn 1
 number 100

ephone-dn 2
 number 101

ephone-dn 3
 number 102

ephone-dn 4
 number 103

ephone-dn 5
 number 104

ephone-dn 6
 number 105

ephone-dn 7
 number 106

ephone-dn 8
 number 107

ephone-dn 9
 number 108


ephone 1
 device-security-mode none
 mac-address 000C.CF4D.B8AD
 type 7960
 button 1:2

ephone 2
 device-security-mode none
 mac-address 0030.A31E.4443
 type 7960
 button 1:3

ephone 3
 device-security-mode none
 mac-address 0006.2A50.CB2D
 type 7960
 button 1:4

ephone 4
 device-security-mode none
 mac-address 0010.1199.2C17
 type 7960
 button 1:5

ephone 5
 device-security-mode none
 mac-address 0000.0C9A.A190
 type 7960
 button 1:6

ephone 6
 device-security-mode none
 mac-address 000A.4184.DC7B
 type 7960
 button 1:7

ephone 7
 device-security-mode none
 mac-address 0001.4210.0C4A
 type 7960
 button 1:8

ephone 8
 device-security-mode none
 mac-address 0040.0B69.04C7
 type 7960
 button 1:9

ephone 9
 device-security-mode none
 mac-address 00D0.97C8.7991
 type 7960
 button 1:1

end
write memory
```

---

## 2. L3 SWITCH 3560 — CORE / Sala de Red

> Este switch es el núcleo de la red. Habilita enrutamiento entre VLANs
> y reenvía las solicitudes DHCP al servidor mediante ip helper-address.

```
enable
configure terminal
hostname SW-Core

! === HABILITAR ENRUTAMIENTO ===
ip routing

! === CREAR TODAS LAS VLANs ===
vlan 10
 name Servidores
vlan 50
 name VoIP
vlan 100
 name Direccion
vlan 120
 name Asesoramiento
vlan 130
 name Diseno-SocialMedia
vlan 140
 name Secretaria
vlan 150
 name SalaRed
vlan 200
 name Aula1
vlan 210
 name Aula2
vlan 220
 name Aula3

! === INTERFAZ ENRUTADA HACIA ROUTER 2811 ===
interface GigabitEthernet0/1
 description Enlace-Router2811
 no switchport
 ip address 172.20.0.1 255.255.255.252
 no shutdown

! === SVIs — GATEWAYS POR VLAN ===
! DHCP RELAY: ip helper-address reenvía solicitudes DHCP al servidor

interface Vlan10
 description Servidores
 ip address 172.20.10.1 255.255.255.0
 no shutdown

interface Vlan50
 description VoIP
 ip address 172.20.50.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan100
 description Direccion
 ip address 172.20.100.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan120
 description Asesoramiento
 ip address 172.20.120.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan130
 description Diseno-SocialMedia
 ip address 172.20.130.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan140
 description Secretaria
 ip address 172.20.140.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan150
 description SalaRed
 ip address 172.20.150.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan200
 description Aula1
 ip address 172.20.200.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan210
 description Aula2
 ip address 172.20.210.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

interface Vlan220
 description Aula3
 ip address 172.20.220.1 255.255.255.0
 ip helper-address 172.20.10.10
 no shutdown

! === PUERTOS TRUNK HACIA ACCESS SWITCHES ===
interface FastEthernet0/1
 description Trunk-SW-Direccion
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 50,100
 no shutdown

interface FastEthernet0/2
 description Trunk-SW-Consulta
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 50,120
 no shutdown

interface FastEthernet0/3
 description Trunk-SW-Diseno
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 50,130
 no shutdown

interface FastEthernet0/4
 description Trunk-SW-Secretaria
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 50,140
 no shutdown

interface FastEthernet0/5
 description Trunk-SW-Aula1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 200
 no shutdown

interface FastEthernet0/6
 description Trunk-SW-Aula2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 210
 no shutdown

interface FastEthernet0/7
 description Trunk-SW-Aula3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 220
 no shutdown

! === PUERTOS ACCESO — SALA DE RED ===
interface FastEthernet0/20
 description Servidor-Espanito
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet0/21
 description PC-SalaRed
 switchport mode access
 switchport access vlan 150
 no shutdown

interface FastEthernet0/22
 description Telephone-SalaRed
 switchport mode access
 switchport access vlan 150
 switchport voice vlan 50
 no shutdown

interface FastEthernet0/23
 description AP-SalaRed
 switchport mode access
 switchport access vlan 150
 no shutdown

! === RUTA POR DEFECTO HACIA ROUTER ===
ip route 0.0.0.0 0.0.0.0 172.20.0.2

end
write memory
```

---

## 3. SW-Direccion — 2960 (Dirección)

> Dispositivos: 1 PC, 1 Laptop, 1 IP Phone, 1 Printer, 1 AP (para móvil)
> VLANs: 100 (datos), 50 (voz)

```
enable
configure terminal
hostname SW-Direccion

! === VLANs ===
vlan 50
 name VoIP
vlan 100
 name Direccion

! === UPLINK TRUNK HACIA L3 SWITCH ===
interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 50,100
 no shutdown

! === PC DIRECCIÓN ===
interface FastEthernet0/1
 description PC-Direccion
 switchport mode access
 switchport access vlan 100
 no shutdown

! === LAPTOP DIRECCIÓN ===
interface FastEthernet0/2
 description Laptop-Direccion
 switchport mode access
 switchport access vlan 100
 no shutdown

! === IMPRESORA DIRECCIÓN ===
interface FastEthernet0/4
 description Impresora-Direccion
 switchport mode access
 switchport access vlan 100
 no shutdown

! === IP PHONE — datos (VLAN 100) + voz (VLAN 50) ===
! El teléfono IP pasa el tráfico del PC en VLAN 100 (data)
! El propio teléfono usa VLAN 50 (voice)
interface FastEthernet0/3
 description IPPhone-Direccion
 switchport mode access
 switchport access vlan 100
 switchport voice vlan 50
 no shutdown

! === ACCESS POINT (móvil dirección se conecta vía WiFi) ===
interface FastEthernet0/5
 description AP-Direccion
 switchport mode access
 switchport access vlan 100
 no shutdown

end
write memory
```

---

## 4. SW-CONSULTA — 2960 (Asesoramiento)

> Dispositivos: 4 PCs, 4 IP Phones, 2 Printers, 4 móviles (vía AP)
> VLANs: 120 (datos), 50 (voz)

```
enable
configure terminal
hostname SW-Consulta

! === VLANs ===
vlan 50
 name VoIP
vlan 120
 name Asesoramiento

! === UPLINK TRUNK HACIA L3 SWITCH ===
interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 50,120
 no shutdown

! === PCs + IP PHONES (Fa0/2 a Fa0/5) ===
! Cada asesor tiene PC + IP Phone enchufado al mismo puerto
interface FastEthernet0/1
 description PC-Asesor1
 switchport mode access
 switchport access vlan 120
 no shutdown

interface FastEthernet0/2
 description PC-Asesor2
 switchport mode access
 switchport access vlan 120
 no shutdown

interface FastEthernet0/3
 description PC-Asesor3
 switchport mode access
 switchport access vlan 120
 no shutdown

interface FastEthernet0/4
 description PC-Asesor4
 switchport mode access
 switchport access vlan 120
 no shutdown

! === IMPRESORAS ===
interface FastEthernet0/5
 description Impresora-Consulta-1
 switchport mode access
 switchport access vlan 120
 no shutdown

interface FastEthernet0/6
 description Impresora-Consulta-2
 switchport mode access
 switchport access vlan 120
 no shutdown

! === IP Phones ===

interface FastEthernet0/7
 description IPPhone-Asesor1
 switchport mode access
 switchport access vlan 120
 switchport voice vlan 50
 no shutdown


 interface FastEthernet0/8
 description IPPhone-Asesor2
 switchport mode access
 switchport access vlan 120
 switchport voice vlan 50
 no shutdown


 interface FastEthernet0/9
 description IPPhone-Asesor3
 switchport mode access
 switchport access vlan 120
 switchport voice vlan 50
 no shutdown

interface FastEthernet0/10
 description IPPhone-Asesor4
 switchport mode access
 switchport access vlan 120
 switchport voice vlan 50
 no shutdown


end
write memory
```

---

## 5. SW-DISEÑO — 2960 (Diseño y Redes Sociales)

> Dispositivos: 2 PCs, 2 IP Phones, 2 tablets, 2 móviles (vía AP)
> VLANs: 130 (datos), 50 (voz)

```
enable
configure terminal
hostname SW-Diseno

! === VLANs ===
vlan 50
 name VoIP
vlan 130
 name Diseno-SocialMedia

! === UPLINK TRUNK HACIA L3 SWITCH ===
interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 50,130
 no shutdown

! === PCs + IP PHONES ===
interface FastEthernet0/1
 description PC-Diseno1
 switchport mode access
 switchport access vlan 130
 no shutdown

interface FastEthernet0/2
 description PC-Diseno2
 switchport mode access
 switchport access vlan 130
 no shutdown


! === IP Phones ===

interface FastEthernet0/3
 description IPPhone-Diseno1
 switchport mode access
 switchport access vlan 130
 switchport voice vlan 50
 no shutdown


 interface FastEthernet0/4
 description IPPhone-Diseno2
 switchport mode access
 switchport access vlan 130
 switchport voice vlan 50
 no shutdown


end
write memory
```

---

## 6. SW-SECRETARIA — 2960 (Secretaría)

> Dispositivos: 1 PC, 1 TV, 1 IP Phone, 1 móvil (vía AP)
> VLANs: 140 (datos), 50 (voz)

```
enable
configure terminal
hostname SW-Secretaria

! === VLANs ===
vlan 50
 name VoIP
vlan 140
 name Secretaria

! === UPLINK TRUNK HACIA L3 SWITCH ===
interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 50,140
 no shutdown

! === PC SECRETARIA ===
interface FastEthernet0/1
 description PC-Secretaria
 switchport mode access
 switchport access vlan 140
 no shutdown


! === IP PHONE SECRETARIA ===
interface FastEthernet0/2
 description IPPhone-Secretaria
 switchport mode access
 switchport access vlan 140
 switchport voice vlan 50
 no shutdown


end
write memory
```

---

## 7. SW-AULA1 — 2960 (Aula 1)

> Dispositivos: 6 PCs
> VLAN: 200

```
enable
configure terminal
hostname SW-Aula1

vlan 200
 name Aula1

interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 200
 no shutdown

interface range FastEthernet0/1 - 7
 description PCs-Aula1
 switchport mode access
 switchport access vlan 200
 no shutdown

end
write memory
```

---

## 8. SW-AULA2 — 2960 (Aula 2)

> Dispositivos: 6 PCs
> VLAN: 210

```
enable
configure terminal
hostname SW-Aula2

vlan 210
 name Aula2

interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 210
 no shutdown

interface range FastEthernet0/1 - 7
 description PCs-Aula2
 switchport mode access
 switchport access vlan 210
 no shutdown

end
write memory
```

---

## 9. SW-AULA3 — 2960 (Aula 3)

> Dispositivos: 6 PCs
> VLAN: 220

```
enable
configure terminal
hostname SW-Aula3

vlan 220
 name Aula3

interface GigabitEthernet0/1
 description Trunk-L3-Switch
 switchport mode trunk
 switchport trunk allowed vlan 220
 no shutdown

interface range FastEthernet0/1 - 7
 description PCs-Aula3
 switchport mode access
 switchport access vlan 220
 no shutdown

end
write memory
```

---

## 10. SERVIDOR — DHCP + DNS

### 10.1 IP estática del servidor (configurar en Packet Tracer gráficamente)

```
IP Address  : 172.20.10.10
Subnet Mask : 255.255.255.0
Gateway     : 172.20.10.1
DNS Server  : 172.20.10.10
```

### 10.2 Servicio DNS (activar en Services → DNS)

```
DNS Service: ON
Domain: espanito.local

Registros A:
  servidor.espanito.local    → 172.20.10.10
  gateway.espanito.local     → 172.20.0.1
  router-voip.espanito.local → 172.20.0.2
  nas.espanito.local         → 172.20.10.11
```

### 10.3 Pools DHCP (Services → DHCP — uno por VLAN)

```
POOL: VLAN10-Servidores
  Network       : 172.20.10.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.10.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.10.20
  Max Users     : 20

POOL: VLAN50-VoIP
  Network       : 172.20.50.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.50.1
  DNS Server    : 172.20.10.10
  TFTP Server   : 172.20.0.2        ← CRÍTICO: apunta al CME del router
  Start IP      : 172.20.50.100
  Max Users     : 50

POOL: VLAN100-Direccion
  Network       : 172.20.100.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.100.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.100.10
  Max Users     : 50

POOL: VLAN120-Asesoramiento
  Network       : 172.20.120.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.120.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.120.10
  Max Users     : 50

POOL: VLAN130-Diseno
  Network       : 172.20.130.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.130.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.130.10
  Max Users     : 50

POOL: VLAN140-Secretaria
  Network       : 172.20.140.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.140.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.140.10
  Max Users     : 10

POOL: VLAN150-SalaRed
  Network       : 172.20.150.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.150.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.150.10
  Max Users     : 20

POOL: VLAN200-Aula1
  Network       : 172.20.200.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.200.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.200.10
  Max Users     : 10

POOL: VLAN210-Aula2
  Network       : 172.20.210.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.210.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.210.10
  Max Users     : 10

POOL: VLAN220-Aula3
  Network       : 172.20.220.0
  Subnet Mask   : 255.255.255.0
  Default GW    : 172.20.220.1
  DNS Server    : 172.20.10.10
  Start IP      : 172.20.220.10
  Max Users     : 10
```

---

## 11. Tabla completa de direcciones IP

### Dispositivos con IP fija

| Dispositivo | VLAN | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| Router 2811 (Fa0/0) | — | 172.20.0.2 | /30 | — |
| L3 Switch (Gi0/1) | — | 172.20.0.1 | /30 | — |
| Servidor Espanito | 10 | 172.20.10.10 | /24 | 172.20.10.1 |

### Dispositivos con IP por DHCP

| Zona | VLAN | Rango DHCP | Gateway |
|---|---|---|---|
| Dirección — PCs/Laptop | 100 | 172.20.100.10 – .30 | 172.20.100.1 |
| Dirección — Móvil (AP) | 100 | 172.20.100.30 – .40 | 172.20.100.1 |
| Dirección — Impresora | 100 | 172.20.100.50 | 172.20.100.1 |
| Asesoramiento — PCs | 120 | 172.20.120.10 – .13 | 172.20.120.1 |
| Asesoramiento — Impresoras | 120 | 172.20.120.50 – .51 | 172.20.120.1 |
| Asesoramiento — Móviles | 120 | 172.20.120.20 – .23 | 172.20.120.1 |
| Diseño — PCs | 130 | 172.20.130.10 – .11 | 172.20.130.1 |
| Diseño — Tablets/Móviles | 130 | 172.20.130.20 – .23 | 172.20.130.1 |
| Secretaría — PC/TV/Móvil | 140 | 172.20.140.10 – .15 | 172.20.140.1 |
| Sala Red — PC/AP/Móvil | 150 | 172.20.150.10 – .15 | 172.20.150.1 |
| Aula 1 — 6 PCs | 200 | 172.20.200.10 – .15 | 172.20.200.1 |
| Aula 2 — 6 PCs | 210 | 172.20.210.10 – .15 | 172.20.210.1 |
| Aula 3 — 6 PCs | 220 | 172.20.220.10 – .15 | 172.20.220.1 |

### Teléfonos VoIP — VLAN 50 (DHCP con TFTP=172.20.0.2)

| Extensión | Ubicación | IP esperada |
|---|---|---|
| 101 | Dirección
| 102 | Asesor 1
| 103 | Asesor 2
| 104 | Asesor 3
| 105 | Asesor 4
| 106 | Diseño 1
| 107 | Diseño 2
| 108 | Secretaría
| 100 | Sala Red

---

## 12. Comandos de verificación (ejecutar en cada dispositivo)

### En el Router 2811
```
show ip interface brief
show ip route
show telephony-service
show ephone registered
```

### En el L3 Switch 3560
```
show ip interface brief
show vlan brief
show ip route
show interfaces trunk
show ip helper-address
```

### En cada Access Switch 2960
```
show vlan brief
show interfaces trunk
show interfaces status
```

### Pruebas de conectividad
```
! Desde cualquier PC — verificar DHCP y gateway:
ipconfig /all          (en PC de PT — Desktop → IP Configuration)

! Desde CLI del router/switch — ping entre VLANs:
ping 172.20.100.10     (PC dirección)
ping 172.20.120.10     (PC asesoramiento)
ping 172.20.10.10      (servidor)
ping 172.20.50.100     (teléfono VoIP dirección)

! DNS — desde un PC en PT → Web Browser:
http://servidor.espanito.local
```

---

## 13. Orden de configuración recomendado en Packet Tracer

```
1. Colocar y conectar todos los dispositivos según topología
2. Configurar el SERVIDOR (IP fija + DHCP pools + DNS)
3. Configurar el L3 SWITCH 3560 (ip routing + VLANs + SVIs + trunks)
4. Configurar el ROUTER 2811 (interfaces + CME telephony)
5. Configurar cada ACCESS SWITCH 2960 (VLANs + puertos)
6. Verificar que los PCs reciben DHCP (Desktop → IP Configuration → DHCP)
7. Verificar que los IP Phones reciben IP y extensión
8. Probar llamadas entre extensiones
9. Ejecutar comandos de verificación
```

---

## 14. Notas importantes para Packet Tracer

> **DHCP desde servidor:** Todos los dispositivos deben tener seleccionado
> "DHCP" en Desktop → IP Configuration. El ip helper-address en el L3 Switch
> redirige las solicitudes broadcast al servidor 172.20.10.10.

> **VoIP:** Los IP Phones 7960 en PT registran automáticamente si:
> (1) DHCP pool VLAN 50 tiene TFTP = 172.20.0.2
> (2) telephony-service está configurado en el router
> (3) El switch tiene switchport voice vlan 50 configurado

> **Modelo de switch L3:** Usar Cisco 3560 (no 2960). En PT: 3560 tiene
> soporte para "ip routing" y SVIs.

> **Trunk en 3560:** Usar "switchport trunk encapsulation dot1q" antes de
> "switchport mode trunk" (requerido en 3560, no en 2960).
