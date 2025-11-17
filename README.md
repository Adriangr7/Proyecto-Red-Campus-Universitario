# 🏫 Topología de Campus Universitario — Diseño y Configuración Completa

Este proyecto consiste en el diseño y configuración integral de una red de campus universitario, basada en una arquitectura jerárquica de tres niveles: Core, Distribución y Acceso. El entorno incluye VLANs, OSPF, HSRP, EtherChannel, STP, seguridad avanzada, WLC, DHCP, NTP, servidores, políticas de red y salida a Internet mediante NAT.

La topología representa un campus con tres edificios (A, B y C), un Data Center, un router de borde, infraestructura WiFi redundante y servicios corporativos internos. Los edificios y su arquitectura interna están diseñados con loops, siguiendo buenas prácticas de resiliencia y redundancia.

---

## 🥅 Objetivos del Proyecto

- Diseñar una arquitectura de red escalable, modular y funcional.  
- Implementar movilidad WiFi entre edificios.  
  - La idea original era mantener una misma SSID mediante Override.  
  - Packet Tracer no soporta esta función, por lo que se implementan **tres WLC** (uno por edificio) simulando un diseño equivalente.  
- Garantizar alta disponibilidad usando HSRP, EtherChannel y STP.  
- Implementar un esquema de direccionamiento IPv4 unificado (**10.0.0.0/8**).  
- Centralizar servicios: **DHCP, NTP, Syslog, DNS, FTP/TFTP, RADIUS, Web**.  
- Separación de SSID por VLAN y arquitectura WiFi controlada por WLC.  
- Implementar OSPF como protocolo de enrutamiento interno.  
- Aplicar un conjunto completo de medidas de seguridad en switches, routers y WLC.

---

## 🏛 Arquitectura de Red

| Capa | Función |
|------|---------|
| **Core / Spine** | Conectividad de alta velocidad, conmutación L3, resiliencia. |
| **Distribución / Leaf** | Agregación de edificios, políticas L3, HSRP, EtherChannel. |
| **Acceso** | Conexión de usuarios, APs, VoIP, PCs y dispositivos finales. |
| **Data Center** | Arquitectura Spine & Leaf con los servicios corporativos. |

Se implementan **tres WLC**, uno por edificio, junto con un conjunto de servidores en el Data Center.

---

## 🗺 Esquema General de VLANs

| VLAN | Uso | Máscara |
|------|------|---------|
| 10 | Alumnos | /22 |
| 20 | Profesores | /22 |
| 30 | Administración | /24 |
| 40 | Laboratorios | /22 |
| 50 | Invitados | /22 |
| 60 | VoIP | /24 |
| 70 | Seguridad | /24 |
| 90 | Biblioteca | /24 |
| 99 | Gestión | /24 |

Cada edificio implementa estas VLANs utilizando rangos dedicados dentro de **10.0.0.0/8**.

---

## 🏢 Direccionamiento por Edificios (Ejemplo Edificio A)

- VLAN 10 → 10.0.8.0/22  
- VLAN 20 → 10.0.20.0/22  
- VLAN 30 → 10.0.30.0/24  
- VLAN 40 → 10.0.40.0/22  
- VLAN 50 → 10.0.48.0/22  
- VLAN 60 → 10.0.60.0/24  
- VLAN 70 → 10.0.70.0/24  
- VLAN 90 → 10.0.90.0/24  
- VLAN 99 → 10.0.99.0/24  

Cada VLAN tiene:  
✔ IP Virtual HSRP (Ejemplo: IP virtual HSRP -- 10.0.11.250 )
✔ Dirección destinada al WLC si la VLAN aloja SSID 

---

## 🔗 EtherChannel y Enlaces /30

Los enlaces L3 utilizan redes /30 agrupadas por bloques.

**Ejemplos:**
- SWE11–SWC1 → 10.250.1.1 /30  
- SWE11–SWC2 → 10.250.1.5 /30  
- Spine1–Leaf1 → 10.250.6.1 /30  



---

## 🛰 Routing — OSPF Área 10
Toda la red opera bajo:
router ospf 10


El Core redistribuye rutas hacia/desde el Data Center y el router de borde.

---

## 🛡 Alta Disponibilidad

- ✔ HSRP en todas las VLAN de distribución  
- ✔ EtherChannel LACP en enlaces críticos  
- ✔ RSTP (Rapid PVST+)  
- ✔ Tres WLC para redundancia WiFi  

---

## 📡 WiFi con WLC

Tres WLC (uno por edificio).  
Credenciales:

- Usuario: `admin`  
- Password: `Modoadmin`

Los SSID se asignan mediante **AP Groups**, cada uno asociado a la VLAN correspondiente del edificio.

---

## 🕒 Servicios Centrales en el Data Center

| Servicio | Subred |
|----------|---------|
| DHCP | 10.250.7.0/30 |
| NTP | 10.250.8.0/30 |
| Syslog | 10.250.9.0/30 |
| DNS | 10.250.10.0/30 |
| FTP/TFTP | 10.250.11.0/30 |
| RADIUS | 10.250.12.0/30 |
| Web | 10.250.13.0/30 |

---

## 🔧 Elementos Implementados

- VTP  
- OSPF 
- HSRP 
- VSIs 
- STP con prioridades 
- CDP/LLDP 
- DTP → Deshabilitado por seguridad
- EtherChannel
- Seguridad (DHCP Snooping, DAI, PortSecurity)
- NAT Overload
- Equipamiento completo en los switches (PCs, VoIP, APs, routers, servidores)

---

## 🔒 Seguridad Implementada

### **DHCP Snooping + DAI**
- Puertos usuario: **UNTRUST**  
- Uplinks: **TRUST**

### **Port-Security**
- 1 MAC por puerto  
- Modo **restrict**  
- Sticky habilitado  

### **AAA + RADIUS**
- Servidor ubicado en el Data Center e integrado en WLCs.
- Solo los WLC utilizan el servidor RADIUS para autenticación WiFi 802.1X.
- Packet Tracer no soporta Mobility Groups, por lo que se simula el roaming con WLC separados.

### **SSH Only**
- Usuario: `admin / modoadmin`  
- Privilegiado: `tunoentras`

### **Banner**
ACCESO RESTRINGIDO

### **Seguridad en Consola**
line console 0
password tunoentras
login


---

## 🌐 Internet Connectivity & NAT Overload

La red accede a Internet a través de un Router de Frontera, donde se implementa:

NAT Overload (PAT) para toda la red interna 10.0.0.0/8

Traducción estática de servicios alojados en el Data Center

Políticas de seguridad perimetral

Esto permite que todo el campus navegue hacia Internet usando una única dirección pública.

---

## 🔧 STP – Diseño sin Bucles

✔ STP solo se ejecuta en enlaces L2 entre switches de Acceso y Distribución.
❌ No se ejecuta en ningún enlace L3 (Core–Distribución, Spine–Leaf, etc.).

Tecnología usada:
Rapid PVST+

---

## 🚀 Futuras Mejoras

- Telefonía IP avanzada  
- ACLs por VLAN  
- Integración de puertos SFP+  
- Mejora del roaming WiFi  
- Redundancia completa del Data Center  

---

## 📦 Conclusión

El diseño final proporciona una red de campus universitaria robusta, modular, segura y escalable, utilizando tecnologías profesionales de Cisco. Incluye redundancia completa, arquitectura Spine–Leaf, políticas de seguridad, movilidad WiFi simulada, servicios centralizados y un plan de direccionamiento avanzado.
