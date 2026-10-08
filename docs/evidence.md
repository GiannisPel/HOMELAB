# Sanitized Evidence

## DNS 

| Evidence | Relevant observation |
|---|---|
| Creation wizard | CT 100; unprivileged; nesting selected; 5 GiB disk; 1000 MiB RAM / 512 MiB swap |
| Network | `vmbr0`; static `.5/24`; gateway `.1`; no VLAN tag |
| OS DNS | Inherited in wizard; correction later discussed but final value unverified |
| Direct resolution | A/AAAA lookups succeeded; AAAA records are not proof of IPv6 routing |
| Router preflight | dnsmasq 2.93; one instance; DHCP start 21, limit 234, lease 12h |
| AdGuard upstream | `https://dns10.quad9.net/dns-query` |
| Query log | Router-originated requests appeared; personal query history excluded |

Running/restored test reconstruction:

```text
query[A] example.net from <LAB_CLIENT>
forwarded example.net to 192.0.2.5
reply example.net is 0.0.0.0
```

Stopped-container test: first query went to AdGuard without an observed answer; client retry about three seconds later went to `1.1.1.1` and received public A records. The client displayed one timeout before success. Router caches were cleared between scenarios. Whole-host shutdown and uninterrupted recovery were not separately tested.

## HP installer

| Observation | Sanitized value |
|---|---|
| Hardware | HP EliteDesk 705 G4 DM; Ryzen 5 PRO 2400G; 8 GB DDR4 |
| Storage | Nominal 256 GB NVMe; 238.47 GiB displayed |
| Disk choices | ext4; hdsize 238.0; other displayed fields automatic |
| Interface | Wired `r8169`; alternate Wi-Fi `iwlwifi` |
| Network | `.4/24`; gateway/DNS `.1`; pinned interface names selected |
| Management login | Successful after an initial 401 error |

Full drive identifiers, MACs and original hostnames are excluded.

## Version and memory snapshots

Both hosts reported after updates:

```text
pve-manager/9.2.21/4f6e0ac86f9e8c7f
running kernel: 7.0.14-22-pve
```

| Host / checkpoint | Total RAM | Used | Available | Swap total / used |
|---|---:|---:|---:|---:|
| HP after update | 6.7 GiB | 1.6 GiB | 5.1 GiB | 6.7 GiB / 0 B |
| VAIO before PDM | 3.8 GiB | 1.6 GiB | 2.2 GiB | 4.0 GiB / 0 B |
| VAIO after guest creation | 3.8 GiB | 1.6 GiB | 2.1 GiB | 4.0 GiB / 0 B |
| VAIO with PDM running | 3.8 GiB | 1.8 GiB | 2.0 GiB | 4.0 GiB / 32 KiB |

Latest VAIO output also showed 829 MiB free and 1.5 GiB buff/cache. An earlier storage snapshot showed `local` 20.05% used and `local-lvm` 34.28% used. These are checkpoints, not guaranteed current capacity.

| Guest | CPU use | RAM used / cap | Swap use | Storage use |
|---|---:|---|---|---|
| AdGuard / CT 100 | 0.16% | 61.35 MiB / 1000 MiB | 0 B | Not captured |
| Tailscale / CT 102 | 0.06% | 32.11 MiB / 1000 MiB | 0 B | Not captured |
| PDM / CT 200 | 0.34% of one core | 110.06 MiB / 1 GiB | 32 KiB / 512 MiB | 2.52 GiB / 15.58 GiB |

Caps limit memory growth rather than reserving all that physical RAM. These light-activity snapshots do not establish peak requirements.

## PDM dashboard 

| Observation | Recorded result |
|---|---|
| Version / host | PDM 1.1.7 / VAIO alias `pve-core` |
| Guest | CT 200, running, unprivileged |
| Remotes | Two reachable remotes, aliases `core` and `compute` |
| PVE nodes | Two online |
| Guests | No VMs; three running LXCs; one stopped LXC |
| Running services | AdGuard, Tailscale and PDM |
| Stopped guest | Purpose unrecorded |
| Registered PBS instances | None shown |
| Aggregate RAM | 3.424 GiB of 10.745 GiB, around 33% |
| Aggregate storage | 68.259 GiB of 407.454 GiB, around 17% |

Aggregates do not imply pooled resources. Original remote IDs, accounts, addresses and unique IPv6 prefix are omitted.

## LAN TCP scan

| Target role | Fictional address | Open TCP ports shown |
|---|---|---|
| Cudy | `192.0.2.1` | 22, 53, 80, 443 |
| Switch | `192.0.2.2` | 80 |
| VAIO | `192.0.2.3` | 22, 3128 |
| HP | `192.0.2.4` | 22, 111, 3128 |
| AdGuard | `192.0.2.5` | 22, 53, 80 |
| Tailscale | `192.0.2.6` | 22 |
| PDM | `192.0.2.10` | 22, 8443 |

Personal client inventory is omitted. The invocation was not supplied; full TCP coverage, UDP and service-version detection are unverified. Basic service labels are not confirmed software identities. PVE 8006 was used successfully elsewhere despite not appearing in this scan excerpt.

The results establish LAN reachability, not public internet exposure. Actual WAN rules, listener ownership and external reachability remain to be reviewed. No ports were closed for this update.

References: [Nmap port states](https://nmap.org/book/man-port-scanning-basics.html), [port selection](https://nmap.org/book/man-port-specification.html).
