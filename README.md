# Micro Homelab - Network Engineering Journal

A small homelab for learning routing, DNS, switching, virtualization and remote administration. Updated through **8 October 2026**.

**Public, anonymized edition:** Local IPs and hostnames are fictional replacements. `192.0.2.0/24` represents the lab and `198.51.100.0/24` the upstream network. These are documentation-only ranges, not deployment addresses. Substitute the real private addresses before using any example. Public resolver addresses and vendor URLs are retained.

Original screenshots are excluded. Their useful results are preserved in [sanitized evidence](docs/sanitized-evidence.md). See [privacy notes](docs/privacy.md) for the scope of anonymization.

## Current status

| Component or milestone | Recorded state |
|---|---|
| Cudy WR3000 v1 / EU1.0 | OpenWrt 25.12.5 installed; LuCI and SSH accessed |
| ISP uplink | Working 5 GHz station connection through `wwan`, assigned to WAN |
| Lab Wi-Fi | Separate 2.4 GHz LAN AP with its own SSID/password |
| TP-Link ES205G | HTTP management reachable in LAN scan; power-cycle persistence unverified |
| VAIO, alias `pve-core` | Proxmox host; AdGuard, Tailscale and PDM running in separate LXCs |
| HP, alias `pve-compute` | Proxmox installed; management accessed; registered in PDM |
| Both Proxmox hosts | PVE 9.2.21, kernel `7.0.14-22-pve` after updates |
| AdGuard filtering | Direct resolution and blocking through the router demonstrated |
| DNS fallback | Worked with AdGuard stopped after one three-second client timeout |
| DNS recovery | Blocking returned after restarting AdGuard and clearing router cache |
| Tailscale | User reported successful connection; restart/mobile-data tests not separately recorded |
| PDM 1.1.7 | Both independent hosts online in one dashboard |
| Port scan | LAN reachability recorded; public internet exposure unverified |
| Nginx Proxy Manager, media stack, password manager | Planned; no installation demonstrated |
| VLANs, isolated guest Wi-Fi, HA cluster | Not deployed in the recorded setup |

## Hardware and roles

| Device | Hardware | Role |
|---|---|---|
| ISP gateway | Existing ISP router | Upstream internet and Wi-Fi |
| Lab router | Cudy WR3000 v1 | DHCP, DNS forwarding, routing, firewall and Wi-Fi |
| Switch | TP-Link ES205G, five Gigabit RJ45 ports | LAN connectivity and future VLAN learning |
| Core host | Older Sony VAIO, 4 GB installed RAM | Lightweight infrastructure guests |
| Compute host | HP EliteDesk 705 G4 DM; Ryzen 5 PRO 2400G, 8 GB DDR4, 256 GB NVMe | Proxmox capacity for heavier workloads |
| Workstation | Desktop PC | Administration and network experiments |

The VAIO battery powers only the laptop. It does not keep the router, switch or HP running. This is not a high-availability deployment.

## Anonymized address plan

| Role | Documentation address | State |
|---|---|---|
| Upstream network / gateway | `198.51.100.0/24` / `198.51.100.1` | Fictional equivalents |
| Cudy wireless WAN | Upstream DHCP | Routed Wi-Fi uplink |
| Lab network | `192.0.2.0/24` | Fictional equivalent |
| Router | `192.0.2.1` | Client gateway and DNS |
| Switch | `192.0.2.2` | Management |
| VAIO / `pve-core` | `192.0.2.3` | Proxmox HTTPS port 8006 |
| HP / `pve-compute` | `192.0.2.4` | Proxmox HTTPS port 8006 |
| AdGuard / CT 100 | `192.0.2.5` | Preferred filtering resolver |
| Tailscale / CT 102 | `192.0.2.6` | Subnet router |
| Nginx Proxy Manager | `192.0.2.7` | Proposed, not deployed |
| PDM / CT 200 | `192.0.2.10` | HTTPS port 8443 |
| Infrastructure allocation | `.2`–`.20` | Manual addresses |
| DHCP pool | `.21`–`.254` | Start 21, limit 234 |

Clients use Cudy as gateway and DNS. Cudy forwards DNS to AdGuard first and an independent public resolver on fallback. The switch management IP is not the client gateway.

## Physical connection allocation

| Connection | Medium | Allocation |
|---|---|---|
| ISP → Cudy | 5 GHz Wi-Fi | No switch port |
| Cudy LAN → switch | 0.5 m Cat6 | Port 1 |
| HP → switch | Short Cat6; 0.25 m proposed | Port 2 planned |
| VAIO → switch | Ethernet, length unrecorded | Port 3 planned |
| Expansion | Unassigned | Port 4 spare |
| Desktop → switch | Long Cat6; 5 m originally proposed | Port 5 planned |

Hosts are reachable, but a complete final physical port inventory is not recorded. The router uplink currently carries the ordinary LAN, not a deployed VLAN trunk.

## Documentation

- [Architecture and availability](docs/architecture.md)
- [Configuration journal, chapters 01–18](docs/configuration-journal.md)
- [Sanitized observations and tests](docs/sanitized-evidence.md)
- [Privacy notes](docs/privacy.md)
- [Screenshot handling](assets/screenshots/README.md)

References: [OpenWrt WR3000 v1](https://openwrt.org/toh/hwdata/cudy/cudy_wr3000_v1), [routed wireless client](https://openwrt.org/docs/guide-user/network/wifi/connect_client_wifi), [ES205G support](https://support.omadanetworks.com/en/product/es205g/v1/), [PDM documentation](https://pdm.proxmox.com/docs/), [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737).
