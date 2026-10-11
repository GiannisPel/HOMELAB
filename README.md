# Micro Homelab — Network Engineering Journal

A small homelab for learning routing, switching, DNS, virtualization and remote administration. Updated through **11 October 2026**.

**Public, anonymized edition:** Addresses, hostnames and SSIDs are substitutes. `192.0.2.0/24` represents the existing LAN and future Homelab, `198.51.100.0/24` the upstream, and `203.0.113.0/24` Guest. Trusted uses `<TRUSTED_NET>/24` and `<TRUSTED_GW>` placeholders. All four real subnets are distinct `/24` networks. These are not deployment addresses; never derive a real subnet from its VLAN number. Public resolvers and vendor links are retained.

Original screenshots, credentials and configuration exports are excluded. See [privacy notes](docs/privacy.md).

## Resume here

**Guest Wi-Fi is reported working and isolated. Homelab VLAN 10 and Trusted VLAN 20 are staged; endpoint migration has not been demonstrated.** Finish segmentation before choosing additional HP workloads.

1. Read the [session handoff](docs/HANDOFF.md) for the current stopping point.
2. Follow the [VLAN migration runbook](docs/vlan-migration.md) for settings and validation gates.
3. Use the [roadmap](docs/roadmap.md) for future goals.

## Current status

| Component or milestone | Recorded state |
|---|---|
| Cudy WR3000 v1 / EU1.0 | OpenWrt 25.12.5 installed; LuCI and SSH accessed |
| ISP uplink | Working 5 GHz station connection through `wwan`, in WAN zone |
| Main Wi-Fi | Separate 2.4 GHz AP; still attached to logical interface `lan` |
| Guest Wi-Fi | Separate subnet/zone; internet and isolation confirmed by user |
| Guest peer isolation / IPv6 | Full two-client and IPv6 validation not recorded |
| ES205G management | Reachable at selected static address after discovery/reconfiguration |
| ES205G VLANs | 802.1Q enabled; VLANs 10 and 20 tagged on port 1; endpoint ports still VLAN 1 |
| OpenWrt VLANs | LAN shown on `br-lan.1`; Trusted on `br-lan.20` with its own gateway |
| Trusted firewall | INPUT exceptions reviewed; corrections and new FORWARD rules supplied, final verification pending |
| AdGuard dashboard | Port 80 is the working-address candidate; `:3000` caused trouble; check final HTTP/HTTPS URL |
| Server/desktop migration | Not demonstrated; target VLAN design is not yet fully deployed |
| VAIO, alias `pve-core` | Proxmox host; native AdGuard, Tailscale and PDM in separate LXCs |
| HP, alias `pve-compute` | Proxmox installed and registered in PDM; no selected compute workload yet |
| Both PVE hosts | Last reported PVE 9.2.21, kernel `7.0.14-22-pve` on 8 October |
| DNS | Filtering, stopped-container fallback and restoration demonstrated on 6 October |
| Tailscale | Connection reported successful; restart/mobile-data tests not separately recorded |
| PDM 1.1.7 | Both independent hosts online in one dashboard; no PVE cluster |
| NPM / media / password manager | Planned options; not deployed |

“Shown” means supplied evidence was reviewed. “Reported” means user confirmation. “Supplied” or “planned” means guidance without final verification.

## Hardware and roles

| Device | Hardware | Role |
|---|---|---|
| ISP gateway | Existing ISP router | Upstream internet and Wi-Fi |
| Lab router | Cudy WR3000 v1 | DHCP, DNS forwarding, routing, firewall and Wi-Fi |
| Switch | TP-Link ES205G, five Gigabit RJ45 ports | Layer 2 VLAN membership and Ethernet connectivity |
| Core host | Older Sony VAIO, 4 GB installed RAM | Lightweight infrastructure guests |
| Compute host | HP EliteDesk 705 G4 DM; Ryzen 5 PRO 2400G, 8 GB DDR4, 256 GB NVMe | Proxmox capacity for heavier workloads |
| Workstation | Desktop PC | Administration and experiments |

The VAIO battery powers only the laptop. PDM aggregates management; this is not a high-availability deployment.

## Address and service plan

| Role | Documentation address | Interface / service |
|---|---|---|
| Upstream gateway | `198.51.100.1` | Cudy `wwan` uses upstream DHCP |
| Existing LAN → Homelab | `192.0.2.0/24` | Keep the real subnet during migration |
| Cudy LAN / future Homelab | `192.0.2.1` | Currently `lan` on `br-lan.1`; target `br-lan.10` |
| Switch | `192.0.2.2` | HTTP 80; management VLAN decision pending |
| VAIO / `pve-core` | `192.0.2.3` | PVE HTTPS 8006; SSH 22 |
| HP / `pve-compute` | `192.0.2.4` | PVE HTTPS 8006; SSH 22 |
| AdGuard / CT 100 | `192.0.2.5` | DNS TCP/UDP 53; dashboard port to verify |
| Tailscale / CT 102 | `192.0.2.6` | Subnet router hosted on VAIO |
| NPM | `192.0.2.7` | Proposed, not deployed |
| PDM / CT 200 | `192.0.2.10` | HTTPS 8443 |
| Trusted | `<TRUSTED_NET>/24` | `trusted`, `br-lan.20`, gateway/DNS `<TRUSTED_GW>` |
| Guest | `203.0.113.0/24` | `guest`, `br-guest`, gateway/DNS `203.0.113.1` |
| Existing LAN infrastructure / DHCP | `.2`–`.20` / `.21`–`.254` | Keep the allocation for Homelab |
| Trusted and Guest DHCP | Host numbers `.100`–`.249` | Start 100, limit 150; lease 12h proposed/recorded |

Clients use their own Cudy subnet gateway as DNS. Cudy prefers AdGuard upstream with an independent public fallback. DHCP stays on Cudy.

## Physical connections

The latest supplied hand drawing supersedes the earlier planned server-port order.

| Connection | Current allocation | Final target |
|---|---|---|
| ISP → Cudy | 5 GHz Wi-Fi station | Retain until a wired uplink is practical |
| Cudy LAN1 → switch | Port 1; 0.5 m Cat6 | Trunk carrying tagged VLANs 10 and 20 |
| VAIO → switch | Port 2 | Untagged/access VLAN 10, PVID 10 |
| HP → switch | Port 3 | Untagged/access VLAN 10, PVID 10 |
| Expansion | Port 4 spare | Future assignment undecided |
| Desktop → switch | Port 5; long Cat6 | Untagged/access VLAN 20, PVID 20 |

Endpoints have not been shown in the target VLANs. Switch management is currently VLAN 1.

## Documentation index

- [Architecture, DNS and availability](docs/architecture.md)
- [Configuration journal, chapters 01–26](docs/configuration-journal.md)
- [Sanitized observations and tests](docs/sanitized-evidence.md)
- [VLAN migration and firewall runbook](docs/vlan-migration.md)
- [Session handoff](docs/HANDOFF.md)
- [Future goals and completion criteria](docs/roadmap.md)
- [Update history](docs/changelog.md)
- [Privacy notes](docs/privacy.md)
- [Screenshot handling](assets/screenshots/README.md)

No network changes were executed for this documentation update. This package has not been pushed to GitHub.

References: [OpenWrt WR3000 v1](https://openwrt.org/toh/hwdata/cudy/cudy_wr3000_v1), [DSA](https://openwrt.org/docs/guide-user/network/dsa/converting-to-dsa), [ES205G support](https://support.omadanetworks.com/en/product/es205g/v1/), [PDM documentation](https://pdm.proxmox.com/docs/), [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737).

