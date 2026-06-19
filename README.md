# Corporate Network with VPN, ASA Firewall and Layer 2 Security
### Cisco Packet Tracer Lab - Cybersecurity and Network Infrastructure


![configs](https://img.shields.io/badge/configs-Device%20Configs-informational)
![docs](https://img.shields.io/badge/docs-Lab%20Report-blue)
![diagrams](https://img.shields.io/badge/diagrams-Network%20Topology-important)
![packet--tracer](https://img.shields.io/badge/packet--tracer-.pkt%20Lab-blue)
---

## Overview

Full corporate network lab implemented in Cisco Packet Tracer, developed as part of the CET in Cybersecurity and Network Infrastructure (IEFP / Centro de Formacao Profissional de Alcoitao, Portugal).

The project simulates a company infrastructure with a central Datacenter and two branch offices (Lisbon and Porto), integrating perimeter security, secure remote access, and Layer 2 protection technologies.

---

## Network Topology

```
LAN Lisbon (192.168.70.32/27)          LAN Porto (192.168.70.64/27)
     |                                       |
  SW0-LISBON                             SW1-PORTO
     |                                       |
  R3-LISBON ---[IPsec Tunnel]---        R4-PORTO ---[IPsec Tunnel]---
     | Fa0/1 (.201)              \       | Fa0/1 (.205)              \
     |                            \      |                             \
     |                    Fa0/1(.202) Fa1/0(.206)                      |
     |                         R5-DC (VPN Hub)                         |
     |                              | Fa0/0 (.1)                       |
     |                         SW2-DC (VLAN10)                         |
     |                         /    |    \                             |
     |                    Site1   Site2  Site3                         |
     |                   (.3)    (.4)    (.5)                          |
     |                              | Gig0/1                           |
     |                           ASA-FW                                |
     |                    G1/1(.2) | G1/2(.201)                        |
     |                             |                                   |
     |                        ISP-ROUTER                               |
     |                    G0/0(.202) | G0/1(.1)                        |
     |                               |                                  |
     |                Simulated Internet (204.204.204.0/24)             |
     |                Web Server (.10) | External Laptop (.20)          |
     |                                 |                                |
     +---[SSL Clientless VPN via HTTPS]+                                |
                                                                        |
     [All sites reach Internet via NAT on ASA] <-----------------------+
```

---

## Technologies Implemented

| Technology | Description |
|---|---|
| IPsec Site-to-Site VPN | Encrypted tunnels between Datacenter, Lisbon and Porto |
| SSL Clientless VPN | Browser-based remote access via HTTPS, no client required |
| Cisco ASA 5506-X | Perimeter firewall with NAT, ACL and stateful inspection |
| Dynamic NAT (PAT) | Internet access using ASA public IP |
| Centralized DHCP | Single DHCP server in DC with relay agents at branches |
| Port Security | MAC flooding protection on server-facing switch ports |
| DHCP Snooping | Rogue DHCP server prevention on VLAN10 |
| VLAN 10 | Datacenter LAN segmentation |

---

## IP Addressing Plan

### Internal Networks

| Segment | Network | Gateway | Description |
|---|---|---|---|
| Datacenter LAN | 192.168.70.0/27 | 192.168.70.1 | Servers and ASA inside |
| Lisbon LAN | 192.168.70.32/27 | 192.168.70.33 | Lisbon branch users |
| Porto LAN | 192.168.70.64/27 | 192.168.70.65 | Porto branch users |
| Lisbon-DC Tunnel | 192.168.70.200/30 | --- | R3 to R5 point-to-point link |
| Porto-DC Tunnel | 192.168.70.204/30 | --- | R4 to R5 point-to-point link |

### Perimeter and Internet

| Segment | Network | Description |
|---|---|---|
| ASA Outside / ISP | 200.200.200.200/30 | ASA-FW to ISP-ROUTER link |
| Simulated Internet | 204.204.204.0/24 | External network with Web Server |

### Fixed IP Assignments

| Device | Interface | IP Address |
|---|---|---|
| R5-DC | Fa0/0 | 192.168.70.1 |
| ASA-FW | G1/1 (inside) | 192.168.70.2 |
| Site1 (DHCP Server) | Fa0 | 192.168.70.3 |
| Site2 | Fa0 | 192.168.70.4 |
| Site3 | Fa0 | 192.168.70.5 |
| SW2-DC (SVI VLAN10) | Vlan10 | 192.168.70.6 |
| R3-LISBON | Fa0/0 | 192.168.70.33 |
| SW0-LISBON (SVI) | Vlan10 | 192.168.70.35 |
| R4-PORTO | Fa0/0 | 192.168.70.65 |
| SW1-PORTO (SVI) | Vlan10 | 192.168.70.67 |
| R3-LISBON | Fa0/1 | 192.168.70.201 |
| R5-DC | Fa0/1 | 192.168.70.202 |
| R4-PORTO | Fa0/1 | 192.168.70.205 |
| R5-DC | Fa1/0 | 192.168.70.206 |
| ASA-FW | G1/2 (outside) | 200.200.200.201 |
| ISP-ROUTER | G0/0 | 200.200.200.202 |
| ISP-ROUTER | G0/1 | 204.204.204.1 |
| Web Server | Fa0 | 204.204.204.10 |
| External Laptop | Fa0 | 204.204.204.20 |

---

## VPN Site-to-Site Parameters

### IKE Phase 1 (ISAKMP)

| Parameter | Value |
|---|---|
| Encryption | AES 128-bit |
| Hash | SHA-1 |
| Authentication | Pre-Shared Key |
| DH Group | Group 2 (1024-bit) |
| Lifetime | 86400 seconds |

### IPsec Phase 2

| Parameter | Value |
|---|---|
| Transform-Set | VPN-SET |
| Encryption | ESP-AES |
| Integrity | ESP-SHA-HMAC |
| Mode | Tunnel |

### VPN Design Notes

The R5-DC acts as the **hub** in a hub-and-spoke topology. Static routes for branch networks were required on R5-DC to ensure return traffic is correctly matched by the crypto ACLs and encrypted through the appropriate tunnel.

---

## DHCP Configuration

### Pools and Ranges

| Pool | Network | Dynamic Range | Gateway | Excluded (Fixed) |
|---|---|---|---|---|
| POOL-DC | 192.168.70.0/27 | .7 to .30 | 192.168.70.1 | .1 to .6 (infrastructure) |
| POOL-LISBON | 192.168.70.32/27 | .36 to .62 | 192.168.70.33 | .33 (R3), .35 (SW0) |
| POOL-PORTO | 192.168.70.64/27 | .68 to .94 | 192.168.70.65 | .65 (R4), .67 (SW1) |

DHCP server is hosted on Site1 (192.168.70.3). Branch routers R3-LISBON and R4-PORTO use `ip helper-address 192.168.70.3` to forward DHCP requests from branch clients to the central server.

---

## Layer 2 Security (SW2-DC)

### Port Security
- Active on server-facing ports: Fa0/2, Fa0/3, Fa0/19 and Laptop0 port Fa0/1
- Maximum 1 MAC address per port
- Dynamic learning mode: sticky
- Violation action: shutdown

### DHCP Snooping
- Active on VLAN10
- Trusted port: Fa0/5 (uplink to R5-DC only)
- All other ports: untrusted

---

## ASA Firewall Configuration Summary

### Outside ACL (OUTSIDE-IN)

| Rule | Protocol | Source | Destination | Port | Reason |
|---|---|---|---|---|---|
| permit | ICMP | any | any | echo-reply | Allow ping replies |
| permit | ICMP | any | any | unreachable | Allow ICMP errors |
| permit | TCP | any | any | 443 | SSL VPN portal |
| permit | UDP | any | any | 500 (IKE) | VPN negotiation |
| deny | IP | any | any | --- | Block everything else |

### SSL Clientless VPN Users

| Username | Access |
|---|---|
| joao | SSL VPN Portal |
| maria | SSL VPN Portal |
| manuel | SSL VPN Portal |

Portal URL: `https://200.200.200.201`

---

## Repository Structure

```
/
├── README.md
├── docs/
│   └── Lab_Report_VPN.pdf
├── configs/
│   ├── R3-LISBON.txt
│   ├── R4-PORTO.txt
│   ├── R5-DC.txt
│   ├── SW0-LISBON.txt
│   ├── SW1-PORTO.txt
│   ├── SW2-DC.txt
│   ├── ASA-FW.txt
│   └── ISP-ROUTER.txt
├── diagrams/
│   └── topology.png
└── packet-tracer/
    └── lab.pkt
```

---

## Production Best Practices (Not Implemented in Packet Tracer)

**Dedicated Native VLAN**
The native VLAN should be changed to an unused VLAN (e.g., VLAN 999) to prevent VLAN hopping attacks. Not critical in this lab since no active trunks exist, but mandatory in production.

**Per-User SSL VPN Bookmarks**
In production, each user would be assigned an individual group-policy with a url-list, granting access only to their designated server. Packet Tracer does not support `url-list` or `webvpn-attributes` commands.

**Centralized AAA (RADIUS / TACACS+)**
Network device authentication should use a centralized AAA server instead of local accounts on each device.

**NTP and Centralized Logging**
All devices should synchronize time via NTP and forward logs to a Syslog server for event correlation.

**High Availability**
In production, the ASA should run in Active/Standby failover mode, and WAN links should have redundancy with HSRP on branch routers.

---

## Test Results

| Test | Origin | Destination | Result |
|---|---|---|---|
| VPN S2S Lisbon-DC | PC1 (192.168.70.36) | Site1/2/3 | Pass |
| VPN S2S Porto-DC | PC0 (192.168.70.68) | Site1/2/3 | Pass |
| Internet access via NAT | PC1 Lisbon | Web Server (204.204.204.10) | Pass |
| Internet access via NAT | PC0 Porto | Web Server (204.204.204.10) | Pass |
| SSL Clientless VPN | External Laptop | https://200.200.200.201 | Pass |
| HTTP internal access | PC1 Lisbon | http://192.168.70.3 | Pass |
| DHCP Lisbon | PC1 | Site1 DHCP Server | Pass - IP .36 |
| DHCP Porto | PC0 | Site1 DHCP Server | Pass - IP .68 |
| DHCP Datacenter | Laptop0 | Site1 DHCP Server | Pass - IP .7 |
| Port Security SW2-DC | Fa0/1,/2,/3,/19 | --- | Active - 0 violations |
| DHCP Snooping SW2-DC | VLAN10 | --- | Active - Fa0/5 trusted |

---

## Author

Filipe Castanheira
GitHub: [filipecastanheira-rgb](https://github.com/filipecastanheira-rgb)
CET in Cybersecurity and Network Infrastructure - IEFP / Alcoitao - 2026
