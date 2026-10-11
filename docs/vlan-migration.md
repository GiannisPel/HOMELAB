# VLAN Migration and Firewall Runbook

**Checkpoint: 11 October 2026.** This records prepared settings and the next migration stages. It is not a claim that the migration is complete. Address placeholders must be replaced privately before use; see the [address plan](../README.md).

## 1. Design and control points

| Layer / component | Responsibility |
|---|---|
| ES205G | 802.1Q membership, access-port untagging, PVID and management VLAN |
| Cudy bridge | Match VLAN tags on the switch uplink and expose `br-lan.N` devices |
| Cudy logical interfaces | Give each routed subnet its own gateway and DHCP configuration |
| Cudy firewall | Control router INPUT and inter-zone FORWARD |
| Proxmox / service firewalls | Optional peer restrictions where same-VLAN traffic bypasses Cudy |

Tagged membership preserves a VLAN tag on egress. Untagged membership removes it. PVID assigns a VLAN to untagged ingress frames. Ingress checking rejects VLANs the port does not belong to. Management VLAN selects the switch's own management traffic; it does not define client gateway addresses.

## 2. Switch staging — shown

| VLAN | Port 1 uplink | Port 2 VAIO | Port 3 HP | Port 4 spare | Port 5 desktop |
|---|---|---|---|---|---|
| 1 | Untagged | Untagged | Untagged | Untagged | Untagged |
| 10 Homelab | Tagged | Nonmember | Nonmember | Nonmember | Nonmember |
| 20 Trusted | Tagged | Nonmember | Nonmember | Nonmember | Nonmember |
| PVID | 1 | 1 | 1 | 1 | 1 |

802.1Q is enabled, ingress checking is enabled and management VLAN remains 1. The VLAN definitions alone do not move endpoints. Use the switch's separate **Save** control for startup persistence; a completed save/reboot test has not been captured.

## 3. Cudy staging — partly verified

Bridge settings supplied:

| VLAN | Local | LAN1 to switch | LAN2 | LAN3 |
|---|---|---|---|---|
| 1 | Checked | Untagged + primary/PVID | Untagged + primary/PVID | Untagged + primary/PVID |
| 10 | Checked | Tagged | Nonmember | Nonmember |
| 20 | Checked | Tagged | Nonmember | Nonmember |

Local allows router participation in the VLAN. VLAN filtering was instructed to be enabled. Later interface evidence confirms LAN `br-lan.1` and Trusted `br-lan.20`, but the final complete bridge table still needs inspection.

| Logical interface | Current / staged device | Gateway | Target |
|---|---|---|---|
| `lan` | `br-lan.1` | `192.0.2.1/24` | Change device only to `br-lan.10` during cutover |
| `trusted` | `br-lan.20` | `<TRUSTED_GW>/24` | Retain |
| `guest` | `br-guest` | `203.0.113.1/24` | Retain |

The extra logical interface named `homelab` was marked for deletion. Apply was instructed and the user proceeded; verify it is absent. Do not delete bridge VLAN 10 or make another overlapping static gateway interface.

Trusted DHCP was supplied as start 100, limit 150, lease 12h and option `6,<TRUSTED_GW>`. The gateway interface has no extra default gateway configured; `wwan` supplies the router's upstream route. DHCPv4 should be enabled. Verify the supplied disabled RA/DHCPv6/NDP settings before describing the new segment as IPv4-only.

## 4. Firewall rule ledger

Definitions: **INPUT** goes to Cudy itself; **FORWARD** crosses Cudy between networks; **OUTPUT** originates on Cudy. Source/destination describe who starts the session, not the direction of every reply packet.

Trusted zone staging: INPUT Reject, OUTPUT Accept and intra-zone forwarding Reject were shown. Forwarding to WAN only was supplied and needs a saved-state check. No general Trusted → LAN forwarding is intended. Same-Layer-2 peers are not isolated by intra-zone routed forwarding policy.

### Trusted router services — INPUT

| Name | Source | Destination | Protocol / ports | State |
|---|---|---|---|---|
| `Allow-Trusted-DNS` | `trusted`, source IP/port blank | This device; `<TRUSTED_GW>` | TCP + UDP destination 53 | Shown; protocol/family correction pending verification |
| `Allow-Trusted-DHCP` | `trusted`, source UDP 68; source IP blank | This device; destination IP blank | UDP destination 67 | Shown; IPv4 correction pending verification |
| `Allow-Trusted-Cudy-Admin` | `trusted`, source IP/port blank | This device; destination IP blank | TCP 22, 80, 443 | Shown; IPv4 correction pending verification |

All should use **Accept**, **IPv4 only**, and summarize as **Accept input**. DHCP needs broadcast/bootstrap traffic, so do not constrain it to an already leased client IP or a single destination IP. ICMP is not part of these exceptions; use DNS/HTTP/TCP tests rather than treating a rejected ping as conclusive failure.

### Trusted service administration — FORWARD

| Name | Destination zone / IP | Protocol / ports | State |
|---|---|---|---|
| `Allow-Trusted-Proxmox` | `lan`; `192.0.2.3` and `192.0.2.4` as separate entries | TCP 22, 8006 | Instructions supplied; final summary pending |
| `Allow-Trusted-PDM` | `lan`; `192.0.2.10` | TCP 8443 | Instructions supplied; final summary pending |
| `Allow-Trusted-Switch` | `lan`; `192.0.2.2` | TCP 80 | Instructions supplied; management decision may require revision |
| `Allow-Trusted-AdGuard-Admin` | `lan`; `192.0.2.5` | TCP 80 if final URL is HTTP; 443 if HTTPS | Conditional instructions supplied; URL/rule confirmation pending |

All use source zone `trusted`, source IP/port blank, Accept and IPv4 only; summaries should say **Accept forward**. Space-separated port values are used in the UI. `lan` is intentionally reused for Homelab, so these rules follow its device migration.

The earlier scan showed AdGuard TCP 80, and the user reports trouble with `:3000`. Confirm the working browser's final scheme/address before settling the rule. Do not allow 3000 merely because it was used for initial setup.

Client DNS intentionally terminates on Cudy. Cudy's own request to AdGuard is **OUTPUT**; no broad Guest/Trusted → AdGuard DNS forwarding exception is required for this architecture. Tailscale account administration is online; a container SSH permission is a separate requirement, not a local Tailscale web dashboard.

### Guest — keep the working setup

Guest uses its own DHCP/DNS gateway, INPUT Reject, OUTPUT Accept and forwarding to WAN only. Its DNS INPUT exception allows TCP/UDP 53, DHCP allows UDP 68 → 67 with no fixed destination IP, and Guest → ISP documentation subnet `198.51.100.0/24` is rejected through WAN. No Guest → LAN permission is intended.

## 5. Migration gates — pending

Do one stage at a time. Record the old setting before changing it and retain the administration path until the new one works. Keep router/switch configuration backups outside the public repository.

### A. Review preparation

1. Confirm the extra `homelab` interface is absent, while `lan` remains on `br-lan.1` and bridge VLAN 10 still exists.
2. Review the router VLAN table, Trusted DHCP, WAN-only forwarding, INPUT corrections and four administration rules.
3. Resolve the AdGuard web port from the final working URL, and verify its rule says **Accept forward**.

**Gate:** No duplicate subnet/gateway; correct INPUT versus FORWARD; desktop still reaches current administration.

### B. Test one Trusted wireless device

Keep desktop wired on VLAN 1. Move the main AP's network mapping from `lan` to `trusted`, leaving Guest untouched. Disable VPN and mobile data on the test device while checking the path.

Verify a Trusted lease in host range `.100`–`.249`, correct gateway/DNS, fresh DNS resolution, internet access, Cudy administration, both PVE panels, PDM, switch management and AdGuard's dashboard.

**Gate:** Actual Trusted access succeeds. If it fails, keep wired administration and restore the AP's prior `lan` mapping while diagnosing. No server cutover yet.

### C. Move desktop to Trusted

On switch port 5, select untagged VLAN 20, PVID 20 and remove VLAN 1 membership. Renew desktop DHCP and repeat management/internet tests.

**Gate:** Wired and wireless Trusted management work. If the wired path fails, use the tested wireless path to restore port 5's previous untagged VLAN 1/PVID 1 settings.

### D. Decide switch management

- **Homelab VLAN 10, same address:** Keeps the original address plan, but servers in the same VLAN can reach management without Cudy filtering.
- **Trusted VLAN 20, new address:** Places management with Trusted, but needs a new IP/gateway and updated switch rule/links.

Check the return gateway. Record the selected option before changing management VLAN. A change to Trusted requires revising the earlier `trusted` → `lan` switch rule; same-Trusted-VLAN access would then bypass Cudy.

### E. Move Homelab

Use the verified Trusted management path. Coordinate the existing `lan` device change to `br-lan.10`, switch ports 2/3 becoming untagged VLAN 10/PVID 10 with VLAN 1 removed, and the selected switch management migration. A brief interruption of Homelab/DNS is possible while these settings disagree.

| Port / interface | Intended post-cutover state |
|---|---|
| Switch port 1 | Tagged VLANs 10 and 20; temporary VLAN 1 only as needed during transition |
| Switch port 2 | VAIO access VLAN 10, PVID 10 |
| Switch port 3 | HP access VLAN 10, PVID 10 |
| Switch port 4 | Spare; final assignment undecided |
| Switch port 5 | Desktop access VLAN 20, PVID 20 |
| Cudy `lan` | `br-lan.10`, existing Homelab gateway/subnet |
| Cudy `trusted` | `br-lan.20` |
| Main AP | `trusted` |
| Guest AP | `guest` |

Keep current server/container IPs and gateways. For this access-port design, do not add VLAN 10 tags inside Proxmox or current guests. Recheck PDM remotes and Cudy → AdGuard DNS.

**Gate:** Management, DNS and existing services work across the intended paths. If this stage fails, use Trusted to restore the prior LAN device, server-port membership/PVID and management placement consistently; do not reset hardware as the first response.

### F. Enforce and validate final policy

After Trusted access is verified, inspect and restrict Homelab-to-router INPUT, retaining the DNS/DHCP functions actually required. Verify Homelab → WAN and the default rejection of new Homelab → Trusted/Guest sessions. Retire temporary VLAN 1 management/rescue arrangements only after alternate access and backups are proven.

## 6. Completion evidence

| Test | Expected final behavior | Current evidence |
|---|---|---|
| Guest lease, DNS and internet | Works through Guest gateway | User confirmed before VLAN migration |
| Guest → internal/ISP management | Rejected | General isolation reported; exhaustive per-target check pending |
| Guest peer-to-peer | Rejected as intended | Two-client test pending |
| Trusted lease and internet | Works through Trusted gateway | Pending |
| Trusted → router / selected dashboards | Works | Pending live Trusted client test |
| Trusted → other Homelab ports | Rejected unless intentionally allowed | Pending |
| Homelab → Trusted/Guest new sessions | Rejected | Pending |
| Responses to allowed sessions | Work | Pending after cutover |
| PDM → both PVE remotes | Remain connected | Working before cutover; retest pending |
| Cudy → AdGuard / fallback / restoration | Keeps selected DNS behavior | Pre-VLAN test passed; post-cutover test pending |
| Tailscale from mobile data | Intended Homelab routes work | Pending |
| Reboot/save persistence | Intended settings survive | Pending |
| IPv6 paths | Disabled where intended or explicitly controlled | Full validation pending |

Record the actual test, source segment and outcome. An absent ARP-cache entry alone does not prove a host is offline. A LAN scan alone does not prove public exposure.

References: [OpenWrt DSA](https://openwrt.org/docs/guide-user/network/dsa/converting-to-dsa), [firewall configuration](https://openwrt.org/docs/guide-user/firewall/firewall_configuration), [DHCP](https://openwrt.org/docs/guide-user/base-system/dhcp), [ES205G user guide](https://static.tp-link.com/upload/manual/2024/202408/20240813/1910013715_Omada%20Easy%20Managed%20Switch_UG.pdf), [AdGuard configuration](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration), [PDM web interface](https://pdm.proxmox.com/docs/web-ui.html).
