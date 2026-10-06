# Micro Homelab: Build and Configuration Journal

A small homelab in Greece for learning network engineering through OpenWrt, Ethernet switching, VLANs, packet capture and remote access.

## Documentation status

| Milestone | Recorded status |
|---|---|
| OpenWrt installed on the Cudy WR3000 v1 | Confirmed: OpenWrt 25.12.5 and LuCI accessed |
| Separate ISP and homelab subnets | Confirmed: ISP `192.168.1.0/24`; homelab `192.168.2.0/24` |
| Wireless ISP uplink | Confirmed: 5 GHz client connection, `wwan`, WAN firewall zone |
| Homelab Wi-Fi | Confirmed: separate 2.4 GHz access point on LAN |
| Router SSH access | Used during package inspection |
| Ethernet switch and long Cat6 cable | Acquired |
| Switch management address | `192.168.2.2` selected; final settings and persistence need evidence |
| Desktop Ethernet connectivity | Connection plan recorded; address and connectivity checks need evidence |
| Vaio connection and OS | Proxmox at `192.168.2.3/24`; HTTPS administration on port 8006 accessed successfully |
| AdGuard Home LXC | Running at `192.168.2.5`; direct DNS resolution and requests through Cudy demonstrated |
| DNS forwarding prerequisites | Inspected: dnsmasq 2.93, DHCP/DNS settings, AdGuard Quad9 upstream and direct public-DNS reachability |
| Router forwarding and filtering | Passed: desktop query through `.1` forwarded to `.5` and returned the expected blocking address |
| AdGuard stopped: external fallback | Passed: `1.1.1.1` returned public addresses after one three-second client timeout |
| AdGuard restarted: restoration | Passed after router cache clearing: `.5` returned the blocking answer again; test cleanup awaiting confirmation |
| Server services and VLAN segmentation | Planned; not documented as deployed |

## Hardware

| Component | Hardware | Role |
|---|---|---|
| ISP gateway | Existing ISP router | Internet access; upstream Wi-Fi network |
| Homelab router | Cudy WR3000 v1 / EU1.0, OpenWrt 25.12.5 | Routing, DHCP, DNS, firewall and Wi-Fi |
| Switch | TP-Link ES205G, 5 Gigabit RJ45 ports | Ethernet connectivity; future VLAN experiments |
| Compute node | HP EliteDesk 705 G4 DM; Ryzen 5 PRO 2400G, 8 GB RAM, 256 GB NVMe | Purchased; intended for compute and media services |
| Secondary node | Old Sony Vaio laptop, approximately 2009; Proxmox VE | Management at `192.168.2.3/24`; AdGuard DNS responds at `.5`; Tailscale at `.6` remains planned |
| Workstation | Desktop PC | Administration and network experiments |

The switch is being documented in **standalone mode**, using its own web interface. An Omada Controller is an optional separate management system. The Cudy continues to be managed through OpenWrt.

## Network and addressing

| Network or device | Address | Status / purpose |
|---|---|---|
| ISP network | `192.168.1.0/24` | Upstream network |
| ISP gateway | `192.168.1.1` | Upstream default gateway |
| Cudy wireless WAN | DHCP address in `192.168.1.0/24` | `wwan` over 5 GHz Wi-Fi |
| Homelab LAN | `192.168.2.0/24` | Separate routed network |
| Cudy LAN gateway | `192.168.2.1` | DHCP and DNS server for the LAN |
| Switch management | `192.168.2.2` | Assigned management address; persistence test not recorded |
| Vaio Proxmox management | `192.168.2.3/24` | Confirmed: web interface reached at `https://192.168.2.3:8006` |
| Future mini PC server | `192.168.2.4` | Reserved for the compute/server host |
| AdGuard Home container | `192.168.2.5` | Forwarding/filtering, stopped-container fallback and restoration after cache clearing validated |
| Tailscale container | `192.168.2.6` | Assigned service address; deployment planned |
| Infrastructure allocation | `192.168.2.2`–`192.168.2.20` | Reserved for manually assigned addresses |
| Dynamic DHCP pool | `192.168.2.21`–`192.168.2.254` | LAN clients; start 21, limit 234 |

The switch management IP is used to reach its administration page. LAN clients use **the Cudy at `192.168.2.1` as their default gateway**.

## Physical connection plan

| Connection | Cable or medium | Ports |
|---|---|---|
| ISP gateway ↔ Cudy | 5 GHz Wi-Fi | Cudy in client/station mode |
| Cudy ↔ switch | 0.5 m Cat6 | Cudy LAN port → switch port 1 |
| Switch ↔ desktop | Long Cat6 | Switch port 5 → desktop Ethernet port |
| Switch ↔ HP compute node | Planned 0.25 m Cat6 | Switch port 2 |
| Switch ↔ Vaio | Connected; cable length not yet documented | Port 3 is the planned allocation; actual port needs confirmation |
| Future expansion | Unassigned | Switch port 4 |

Router-to-switch and desktop connections initially use the default LAN. A tagged VLAN trunk is a future configuration step, not a completed configuration in this draft.

## Configuration journal

The [configuration journal](docs/configuration-journal.md) covers installation, addressing, the wireless uplink, switch configuration and validation. Each chapter has a place for the corresponding photographs and observations.

The [screenshot guide](assets/screenshots/README.md) gives filenames and caption conventions for adding evidence to the repository.

## References

- [OpenWrt: Cudy WR3000 v1 hardware information](https://openwrt.org/toh/hwdata/cudy/cudy_wr3000_v1)
- [OpenWrt: routed wireless client](https://openwrt.org/docs/guide-user/network/wifi/connect_client_wifi)
- [TP-Link: ES205G v1 documentation](https://support.omadanetworks.com/en/product/es205g/v1/)
- [Microsoft: ipconfig](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig)
