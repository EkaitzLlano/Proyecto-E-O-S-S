
# 🛡️ EOSS — Enterprise Open Source Security

> Stack de seguridad perimetral completo sobre hardware recuperado. Coste total: **154 €**.  
> Validado en producción real y contra firewalls corporativos Palo Alto + FortiGate.

---

## Stack técnico

| Capa | Tecnología | Función |
|---|---|---|
| **Firewall / OS** | OPNsense 26.1.7 · FreeBSD 14.3 | Firewall perimetral, routing, VLANs, NAT |
| **IPS** | Suricata 7 · Netmap · Hyperscan | Inspección de paquetes en línea (29.236 reglas drop) |
| **DPI** | Zenarmor (Sensei) v2.5 | Inspección de aplicaciones capa 7 sobre LAN |
| **Threat Intel** | CrowdSec + bouncer pf | Bloqueo colaborativo en tiempo real |
| **Detección** | MalTrail | Detección de tráfico malicioso por IPs/dominios |
| **DNS** | AdGuard Home + Unbound DNS | Doble capa filtrado DNS (1.920.000 + 2.002.517 reglas) |
| **VPN** | WireGuard (wg0/wg1) | Acceso remoto cifrado, dos perfiles de acceso |
| **Proxy** | Shadowsocks Blake3/AES-256 | Evasión DPI, encapsulado sobre WireGuard |
| **Monitorización** | Grafana · Prometheus · Loki · Promtail | Dashboards, métricas y logs centralizados |
| **Disponibilidad** | Uptime Kuma + Telegram | Alertas inmediatas ante caídas de servicio |
| **Gestión** | Portainer · Cockpit | Docker y administración del SO |

---

## Hardware

```
┌─────────────────────────────────────────────────────────────┐
│  Nodo firewall    HP ProDesk SFF · i5-8500 (6c) · 12 GB    │
│  Monitorización   Intel NUC10i3FNK · i3-10110U · 16 GB     │
│  DNS              Raspberry Pi 3B V1.2 · ARM A53 · 1 GB    │
│  Switch           MikroTik SwitchOS · 802.1Q VLAN Tagging  │
│  Estación         CachyOS · Ryzen 7 9800X3D · 32 GB        │
└─────────────────────────────────────────────────────────────┘
  Todo el hardware recuperado de residuos empresariales.
  Coste total: 154 € (switch + tarjeta de red Intel I225)
```

---

## Topología de red

```
INTERNET (ISP Orange)
        │
  Router Livebox  ── 192.168.1.1
        │ DMZ
  ┌─────▼──────┐
  │  OPNsense  │  WAN: 192.168.1.10 / LAN: 10.0.0.1
  └─────┬──────┘
        │ 802.1Q trunk
  ┌─────▼──────┐
  │  MikroTik  │  10.0.0.2
  └──┬───┬───┬─┘
     │   │   │
  VLAN10  VLAN20  VLAN1
  10.10.10.0/24  10.20.20.0/24  (gestión)
  Servidores     Equipos

  VPN_LAN      wg0  10.50.50.0/24  (acceso completo)
  VPN_LIMITED  wg1  10.60.60.0/24  (solo internet)
  Shadowsocks  SOCKS5  172.x.x.x  (evasión DPI)
```

---

## Métricas en producción

| Métrica | Valor |
|---|---|
| Reglas Suricata activas (modo drop) | 29.236 |
| Bloqueos CrowdSec — semana 1 | 93.700 IPs |
| Bloqueos CrowdSec — semana 2 | 223.000 IPs (+138%) |
| Consultas DNS en 90 días (AdGuard) | 2.151.439 |
| Tasa de bloqueo DNS (AdGuard) | 7,41% |
| Tasa de bloqueo DNS (Unbound Protect All) | 34,47% |
| Uptime stack completo — 30 días | 99,92% |
| CPU firewall en operación normal | ~3,5% (idle 96%) |
| Temperatura firewall bajo carga | 42 °C |
| Temperatura ambiente núcleos WireGuard | 25 °C |

---

## Validación corporativa — Shadowsocks vs. Palo Alto + FortiGate

Durante las prácticas en empresa (Ekiolan Ingeniería, cliente Exkal):

- La cadena **WireGuard + Shadowsocks** cruzó una VLAN de invitados corporativa y dos firewalls enterprise en producción (**Palo Alto FW-PASeries + FortiGate FW-FortiGate**) sin ser bloqueada.
- El SOC de **S21sec** generó dos alertas de severidad **Alta** (IDs 4154702 y 4154705) clasificando el tráfico Shadowsocks como *"Connection to a TOR Node (Thales Intelligence)"* — el cifrado era tan opaco que la inteligencia de amenazas lo confundió con tráfico TOR.
- Táctica MITRE ATT&CK: **TA0011** (Command and Control) / **T1071.001** (Application Layer Protocol: Web Protocols).
- Ambas alertas cerradas: *"Incident without impact"*.

> Esto valida que el stack doméstico de 154 € supera en capacidad de evasión a la inspección DPI de firewalls corporativos de gama alta.

---

## Prueba IPS — inspección de archivo malicioso en tránsito

- Descarga de archivo RAR con contenido malicioso simulado desde la estación CachyOS.
- Suricata (modo Netmap IPS) **interceptó y bloqueó la descarga al 99%** antes de completarse.
- Overhead durante la prueba: +15% CPU pico, +100 MB RAM — carga asumible.
- Capacidad no disponible en FortiGate 40F (~2.100 €) ni Sophos XGS 107 (~1.470 €) en su rango de precio.

---

## Comparativa económica (3 años)

| Solución | Coste total estimado |
|---|---|
| **EOSS (este proyecto)** | **~1.053 €** |
| Sophos XGS 107 | ~3.822 € |
| Fortinet FortiGate 40F | ~5.352 € |

*Incluye hardware, licencias y soporte. EOSS: hardware recuperado + 0 € en licencias.*

---

## Tecnologías y licencias

```
OPNsense / HardenedBSD  BSD 2-Clause     WireGuard        GPL v2
Suricata                GPL v2           CrowdSec         MIT / Apache 2.0
MalTrail                MIT              AdGuard Home     GPL v3
Grafana                 AGPL v3          Prometheus       Apache 2.0
Loki / Promtail         AGPL v3          Portainer CE     zlib
Cockpit                 LGPL v2.1        Uptime Kuma      MIT
Shadowsocks             Apache 2.0       nginx            BSD 2-Clause
```

**Coste total en licencias: 0 €**

---

## Contexto

Proyecto de fin de ciclo — **2º GRADO SUPERIOR ASIR** (Administración de Sistemas Informáticos en Red).  
Centro: **CIP Tafalla** (Navarra) · Período: noviembre 2025 – mayo 2026.  
Alumno: **Ekaitz Llano Alonso**

---

## Contacto

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Ekaitz_Llano-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/ekaitz-llano)
[![Email](https://img.shields.io/badge/Email-contacto-D14836?style=flat&logo=gmail)](mailto:tu@email.com)

---

<sub>⚠️ La documentación técnica completa (memoria TFG) no es pública por contener datos de red doméstica y referencias a incidentes de seguridad en entornos de terceros.</sub>