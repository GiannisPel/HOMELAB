# Configuration Journal — Anonymized Edition

Updated through **11 October 2026**. Chapters 01–13 preserve the chronology of the 6 October setup; chapters 14–18 cover the subsequent host/service work; chapters 19–26 record Guest and VLAN progress through 11 October. Each chapter is a historical checkpoint. Recommendations and unverified checks are not presented as completed work.

**Privacy:** All local IPv4 addresses and hostnames are fictional substitutions. `192.0.2.0/24` represents the lab; `198.51.100.0/24` represents the upstream; `203.0.113.0/24` represents Guest. Trusted uses `<TRUSTED_NET>/24` and `<TRUSTED_GW>` placeholders. These are documentation-only values: replace them with the actual private addresses before using any example. Public resolver IPs and vendor URLs are retained. Original screenshots are excluded; see [sanitized evidence](sanitized-evidence.md) and [privacy notes](privacy.md).

Command excerpts record earlier guidance, not new actions performed during this documentation update. Image observations are anonymized transcriptions, not edited screenshots.

## 01 — Hardware identification

**Goal:** Record the exact devices used in the build.

The router was identified as **Cudy WR3000 v1**, with an **EU1.0** label. The compute node is an **HP EliteDesk 705 G4 DM**. The selected Ethernet switch is a **TP-Link ES205G**.

**Evidence not fully recorded:** device model labels, switch hardware revision, HP specifications, and the physical build. Hardware models can be recorded as text; source photographs remain outside the public archive.

## 02 — OpenWrt installation

**Recorded outcome:** LuCI became accessible after installation, and the status page showed **OpenWrt 25.12.5** on **Cudy WR3000 v1**.

During the firmware transition, a browser tab continued showing a flashing/reboot message. A fresh incognito session reached LuCI. This observation showed that the old tab was not reliable evidence that flashing was still in progress.

**Evidence not fully recorded:** firmware filenames and source URLs, upload confirmation, LuCI login, and the final status page. Exact image checksums and the intermediate firmware version have not been recorded here.

This chapter records the installation result; it is not yet a complete flashing tutorial.

## 03 — LAN addressing and administration

**Goal:** Keep the homelab LAN separate from the upstream ISP network.

| Setting | Recorded value |
|---|---|
| Cudy LAN address | `192.0.2.1` |
| Subnet mask | `255.255.255.0` |
| DHCP start | `21` |
| DHCP limit | `234` |
| Resulting dynamic range | `192.0.2.21`–`192.0.2.254` |

LuCI was accessed at the new address. One address-change attempt caused LuCI's connectivity check to roll back the pending configuration; the precise final apply/reconnect sequence was not captured as a complete transcript.

**Evidence not fully recorded:** LAN configuration, DHCP pool, and an administration session at `192.0.2.1`.

## 04 — Wireless ISP uplink

**Recorded outcome:** The Cudy connected to the ISP router over **5 GHz Wi-Fi**.

| Setting | Recorded value |
|---|---|
| Wireless operating mode | Client / station |
| Network interface | `wwan` |
| Address assignment | DHCP client |
| Firewall zone | `wan` |
| Upstream subnet | `198.51.100.0/24` |
| Upstream gateway | `198.51.100.1` |

A separate **2.4 GHz access point** was configured for the homelab LAN, with its own SSID and password. Internet access was reported as working.

**Evidence not fully recorded:** wireless scan results, client association, `wwan` address, the LAN access point and a connectivity result. Password fields should remain obscured in any published image.

## 05 — Router SSH and software inspection

Router SSH access uses:

```sh
ssh root@192.0.2.1
```

The package manager was used to compare installation plans:

```sh
apk update
apk add --simulate luci-proto-wireguard
apk add --simulate tailscale
```

| Simulated installation | New packages | Reported total package size afterward |
|---|---:|---:|
| WireGuard and its LuCI interface | 12 | 18.6 MiB |
| Tailscale | 2 | 43.3 MiB |

These are simulations, **not completed VPN installations**. The totals include existing packages and represent uncompressed package sizes. They are not the individual VPN package sizes or direct measurements of compressed flash usage.

The router's storage page showed approximately **5.6 MiB of free writable flash**. Its much larger temporary filesystem uses RAM; it does not expand persistent package storage.

**Evidence not fully recorded:** memory/storage status and the package simulation output. Never include private keys, authentication keys or passwords in terminal screenshots.

## 06 — Switch management address

**Selected configuration:** Give the switch a manual management address within the infrastructure allocation.

| Setting | Selected value |
|---|---|
| DHCP client | Disabled |
| Management IP | `192.0.2.2` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.0.2.1` |
| DNS server, if exposed by firmware | `192.0.2.1` |

The documented menu is **System Info → IP Settings**. Applying the new address requires reconnecting at `http://192.0.2.2`. If the interface has a separate **Save** button, save the configuration after applying it.

Disabling DHCP here refers to the **switch's DHCP client**. The Cudy's DHCP server continues assigning addresses to LAN clients.

**Later observation:** The 8 October LAN scan found the management HTTP port reachable at the selected address. Final saved settings and persistence across a switch power cycle remain unverified.

## 07 — Desktop Ethernet connection

**Connection plan:** Cudy LAN → switch port 1 using 0.5 m Cat6; switch port 5 → desktop using the long Cat6 cable.

For a clean test, turn off the desktop's Wi-Fi and leave its Ethernet IP/DNS settings on automatic.

```cmd
ipconfig /all
ping 192.0.2.1
ping 192.0.2.2
nslookup example.com
```

| Check | Expected result | Recorded result |
|---|---|---|
| Ethernet IPv4 | An address in `192.0.2.0/24` | Awaiting evidence |
| Subnet mask | `255.255.255.0` | Awaiting evidence |
| Default gateway | `192.0.2.1` | Awaiting evidence |
| Router administration | LuCI opens at `192.0.2.1` | Awaiting Ethernet test |
| Switch administration | Web interface opens at `192.0.2.2` | Awaiting evidence |
| Internet and DNS | A website loads with desktop Wi-Fi off | Awaiting evidence |

The ISP uplink remains wireless even after the desktop and switch are wired.

## 08 — Server nodes

**HP compute node at this stage:** Purchased with Ryzen 5 PRO 2400G, 8 GB RAM and 256 GB NVMe. Chapter 15 records the later Proxmox installation.

**Vaio:** The laptop is running **Proxmox VE** and was connected to the switch on 6 October 2026. The management address was changed to **`192.0.2.3/24`**, and the user successfully accessed **`https://192.0.2.3:8006`**. Later versions and workloads are recorded in chapters 14–17.

Read-only checks for recording the final interface and route configuration are:

```sh
ip -br addr
ip route
cat /etc/network/interfaces
```

In a typical Proxmox installation, the management address is assigned to a Linux bridge such as `vmbr0`. Plugging the host into another network does not necessarily update a manually configured address. Inspect the existing configuration before editing it.

The host previously had the reported address `198.51.100.50` and gateway `198.51.100.0`; the latter is a network address for a `/24`, rather than a valid host gateway. The recommended gateway and DNS on the new LAN are `192.0.2.1`. Successful web access confirms the new management address; the final default route and DNS configuration still need direct evidence.

**Service progress:** AdGuard Home runs at `192.0.2.5`. Direct DNS resolution, forwarding/blocking through Cudy, external fallback with CT 100 stopped, and restoration after CT 100 restart and router cache clearing are demonstrated in chapters 10–13. Whole-Vaio shutdown has not been tested. Tailscale was pending at this stage; chapter 14 records its later deployment. Reverse proxy and dashboards remain possible later workloads.

**Evidence not fully recorded:** specifications, installation screens and final storage/RAM configuration. DNS resolution and the three DNS-path tests are recorded in chapter 13; Later VPN progress is recorded in chapter 14. Jellyfin and file syncing remain planned options.

## 09 — AdGuard Home container creation

**Wizard evidence:** Five supplied images showed LXC creation settings. The wizard alone did not prove installation or startup; later DNS tests establish the running service. Template and CPU selections were not visible. Raw images are excluded from this archive.

| Setting | Observed in screenshots | Recommended next setting |
|---|---|---|
| Node | `pve-core` | Keep |
| CT ID | `100` | Keep if available |
| Hostname | `AdGuard` | `adguard` for consistent naming |
| Unprivileged | Selected | Keep |
| Nesting | Selected | Retain the selected setting; modern Debian/systemd workloads can require nesting even without Docker |
| Storage / disk | `local-lvm`, 5 GiB | Keep; monitor log growth |
| RAM / swap limit | 1000 MiB / 512 MiB | 512 MiB RAM is a starting allocation; 1000 MiB is also valid |
| Network bridge | `vmbr0` | Keep |
| IPv4 address | `192.0.2.5/24` | Keep: user-assigned AdGuard Home address |
| IPv4 gateway | `192.0.2.1` | Keep |
| VLAN tag | None | Keep for the current unsegmented LAN |
| Firewall flag | Selected | Keep; actual filtering depends on Proxmox firewall configuration |
| Container DNS | Inherit host settings | Explicit independent resolver, such as `1.1.1.1`, for this DNS-service container |

The container DNS field controls name resolution for the container's own OS. It does not configure the DNS addresses advertised to LAN clients or the upstream resolvers used by AdGuard Home.

The user confirmed the final address plan on 6 October 2026: router `.1`, switch `.2`, Vaio `.3`, future mini PC server `.4`, AdGuard Home `.5`, and Tailscale `.6`, all within `192.0.2.0/24`. The captured AdGuard address matches this plan and should remain `192.0.2.5/24`. AdGuard initial setup uses `http://192.0.2.5:3000`; after selecting web port 80, administration uses `http://192.0.2.5`. Chapter 10 records the later successful DNS tests.

General: CT 100, hostname AdGuard, unprivileged and nesting selected.

Disk: 5 GiB on local-lvm.

Memory: 1000 MiB RAM and 512 MiB swap limit.

Network: vmbr0, static 192.0.2.5/24, gateway 192.0.2.1, no VLAN tag.

DNS: host settings inherited in the wizard.

## 10 — AdGuard Home direct DNS validation

**Recorded outcome:** On 6 October 2026, the AdGuard setup page showed its DNS listener at `192.0.2.5`. The desktop then successfully queried that address for two domains; query names are anonymized in this edition.

The Router tab contains manual device-configuration instructions. Viewing that tab does not apply settings to OpenWrt.

```powershell
nslookup example.com 192.0.2.5
```

| Observation | Recorded result |
|---|---|
| DNS server queried | `192.0.2.5` |
| Two direct domain lookups (names anonymized) | Returned IPv4 and IPv6 addresses without timeout |
| Displayed server name | `UnKnown`; server address remained correctly shown as `.5` |
| Router forwarding / fallback | Not configured or tested in this evidence |
| Ad blocking behavior | Not established by these successful lookups |

The returned IPv6 addresses are DNS records; these tests do not establish working IPv6 connectivity. The displayed `UnKnown` name is consistent with an unavailable reverse DNS name for the local DNS server, rather than a failed forward lookup.

**Next configuration:** Keep LAN clients using Cudy DNS at `192.0.2.1`. Prefer AdGuard at `192.0.2.5` upstream, with an independent external fallback. Inspect the current OpenWrt dnsmasq configuration and validate upstream ordering before applying changes. Test forwarding with AdGuard running, stopped and restarted. Tailscale deployment at `192.0.2.6` follows separately.

## 11 — DNS forwarding prerequisites and reasoning

**Observed on 6 October 2026:** The router's current configuration, dnsmasq build information, direct public-DNS lookup and AdGuard upstream were inspected before changing the lab's DNS path. These checks establish prerequisites; they do not demonstrate configured or tested failover.

| Output | Why inspect it? | Observation and implication |
|---|---|---|
| `uci show dhcp` | Inspect persistent DHCP/DNS configuration and identify settings that must be preserved. | One dnsmasq instance; LAN DHCP start 21, limit 234, lease time 12h. The pool remains `.21`–`.254`, outside the infrastructure allocation. Upstreams currently come from `/tmp/resolv.conf.d/resolv.conf.auto`; no explicit AdGuard upstream or strict-order setting is present. |
| `dnsmasq --version` | Confirm the installed version and compiled features before selecting configuration options. | Version 2.93; DNS and DHCP available, with loop detection compiled. DNSSEC validation and dnsmasq's own DHCPv6 support are not compiled; OpenWrt has a separate odhcpd configuration for IPv6. No package change was made. |
| `nslookup example.com 1.1.1.1` on the Cudy | Test whether the router can contact an independent external resolver, rather than relying only on a configured address. | Successful IPv4 and IPv6 DNS answers. Cloudflare DNS is reachable from the router at the time of this test. Returned AAAA records do not establish IPv6 connectivity. |
| AdGuard upstream settings | Identify where AdGuard sends queries and avoid creating a circular dependency. | The displayed upstream is `https://dns10.quad9.net/dns-query`, a direct DNS-over-HTTPS endpoint. It does not point general queries back to the Cudy. Bootstrap and private reverse-DNS settings are not shown. |

AdGuard upstream: the displayed public upstream is Quad9 over HTTPS.

Router inspection: existing DHCP settings, dnsmasq 2.93 compile options and the direct lookup against `1.1.1.1`.

The completed lookup returned A and AAAA records without a timeout.

**Selected next design, not yet applied:** LAN clients query Cudy at `192.0.2.1`. Cudy prefers AdGuard at `192.0.2.5`, and uses an independent public resolver if AdGuard is unavailable. AdGuard resolves allowed queries through Quad9. General upstream queries must not follow the loop Cudy → AdGuard → Cudy.

Dnsmasq's default upstream selection does not guarantee that AdGuard is preferred. Its documented `strict-order` setting tries servers in resolver-file order. An ordered resolver file avoids assuming that a GUI or command-line server list preserves that order. Default retry behavior relies on client retries, so failover timing requires a live test; instantaneous recovery has not been established.

**Remaining validation:** Apply ordered forwarding; verify a fresh client query appears in AdGuard's query log; stop only the AdGuard container and verify an uncached lookup uses the external fallback; restart AdGuard and verify filtering resumes. Router caches and cached client answers must be accounted for when testing. DHCP remains on Cudy. Tailscale was a subsequent task, recorded in chapter 14.

References: [dnsmasq manual](https://dnsmasq.org/docs/dnsmasq-man.html), [AdGuard Home configuration](https://adguard-dns.io/kb/adguard-home/configuration/).

## 12 — Queries received from the Cudy and command explanations

**Observed on 6 October 2026:** AdGuard's query log shows recent requests for several domains (personal query history omitted). Every displayed client is `192.0.2.1`, the Cudy. Exact activity timestamps are omitted from the public edition.

This establishes that AdGuard receives and processes DNS queries from the router. It is consistent with the selected forwarding design, but the screenshot alone does not distinguish requests made by the router itself from requests forwarded for LAN clients. It does not prove that every device uses this path, that the exact ordered-forwarding commands were applied, that an ad domain was blocked, or that fallback works with AdGuard stopped.

When Cudy forwards clients' queries, AdGuard sees Cudy as the requesting client. Per-device query statistics are therefore aggregated under the router in this design. Entries marked `Processed` are not evidence of a blocking test. `A` denotes an IPv4-address lookup. `HTTPS` denotes a DNS resource-record type containing connection information; it is distinct from DNS-over-HTTPS transport. `Plain DNS` describes the incoming router-to-AdGuard DNS transport, while the selected AdGuard-to-Quad9 upstream uses HTTPS.

### Meaning of the proposed router commands

The commands below were provided for the user to run on Cudy. They are recorded as instructions, not as remotely executed or individually verified commands.

```sh
test -e /etc/config/dhcp.before-adguard ||
    cp /etc/config/dhcp /etc/config/dhcp.before-adguard

cat > /etc/homelab-dns-upstreams.conf <<'EOF'
nameserver 192.0.2.5
nameserver 1.1.1.1
EOF

uci set 'dhcp.@dnsmasq[0].resolvfile=/etc/homelab-dns-upstreams.conf'
uci set 'dhcp.@dnsmasq[0].noresolv=0'
uci set 'dhcp.@dnsmasq[0].strictorder=1'
uci set 'dhcp.@dnsmasq[0].allservers=0'
uci set 'dhcp.@dnsmasq[0].localuse=1'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

| Command or setting | Meaning |
|---|---|
| `test -e ... || cp ...` | Check whether the backup exists; copy the current DHCP/DNS configuration only if it does not. An earlier backup is preserved. |
| `cat > ... <<'EOF'` | Create or overwrite the file with the literal lines up to the closing `EOF`. |
| `nameserver` lines | File contents defining AdGuard first and Cloudflare second, rather than shell commands. |
| `uci set` | Stage a change through OpenWrt's configuration interface. `dhcp` is the configuration file; `@dnsmasq[0]` selects its first dnsmasq section. |
| `resolvfile=...` | Read upstream resolver addresses from the specified custom file. |
| `noresolv=0` | Leave resolver-file reading enabled; zero disables the option named noresolv. |
| `strictorder=1` | Try upstream resolvers in file order. Retry and failure behavior still need a live test. |
| `allservers=0` | Disable sending each query to all upstreams simultaneously. |
| `localuse=1` | Make the router's own default name resolution use its local dnsmasq service. |
| `uci commit dhcp` | Save the staged configuration persistently. |
| `/etc/init.d/dnsmasq restart` | Restart the dnsmasq service to apply the settings; does not reboot the router. |

The desktop test `nslookup example.net 192.0.2.1` explicitly queries Cudy. It does not change or verify the desktop's default DNS setting. To inspect default behavior, run `ipconfig /all` and `nslookup example.net` without a server argument, then check AdGuard's log. Cached router responses might not appear as new upstream requests.

### Coverage of wired and wireless devices

For the current unsegmented LAN, clients obtaining DNS automatically from Cudy should use `192.0.2.1`, whether they connect through the Ethernet switch or the Cudy's LAN Wi-Fi. The router forwards uncached public-domain queries to AdGuard when available. Clients therefore continue to display Cudy as their configured DNS address; AdGuard supplies filtering upstream.

Devices connected directly to ISP Wi-Fi are on the upstream ISP network and are not covered by this configuration. A manually selected external resolver, externally directed secure/private DNS, or VPN-provided DNS may bypass the router's chosen DNS path. Future guest networks or VLANs need their own DHCP/DNS and firewall checks. No DNS-enforcement firewall rules have been applied in this journal.

**Subsequent results:** Chapter 13 records blocking, stopped-container fallback and restored blocking. Default DNS coverage for every wired/wireless client remains unverified. Filtering is unavailable while requests use the public fallback.

References: [OpenWrt DHCP/DNS configuration](https://openwrt.org/docs/guide-user/base-system/dhcp), [dnsmasq manual](https://dnsmasq.org/docs/dnsmasq-man.html), [RFC 9460: HTTPS DNS records](https://www.rfc-editor.org/rfc/rfc9460.html), [Firefox DNS over HTTPS](https://support.mozilla.org/en-US/kb/firefox-dns-over-https).

## 13 — DNS forwarding and failover validation

### A: AdGuard running — passed

**Observed on 6 October 2026:** A desktop query through Cudy returned the blocking address `0.0.0.0` for `example.net`. The router log records the same query from `<LAB_CLIENT>`, forwarding to AdGuard at `192.0.2.5`, and the reply `0.0.0.0`. This validates router forwarding and the expected blocking response for the temporary `||example.net^` rule. The AdGuard query-log view of the matched rule was not supplied in this step.

```powershell
nslookup -type=A -timeout=3 -retry=3 example.net. 192.0.2.1
```

The queried DNS server is `OpenWrt.lan`, address `192.0.2.1`. The answer for `example.net` is `0.0.0.0`.

The relevant request follows this anonymized sequence:

```text
query[A] example.net from <LAB_CLIENT>
forwarded example.net to 192.0.2.5
reply example.net is 0.0.0.0
```

A subsequent repeated lookup is answered from Cudy's cache as `0.0.0.0`. Clear that cache before testing AdGuard shutdown; otherwise a cached blocked answer could conceal the fallback behavior.

### B: AdGuard container stopped — passed with a client retry

In Proxmox, shut down only CT 100 and wait until its status is stopped. Keep Cudy and the Vaio host running. Restart dnsmasq on Cudy to clear the cached blocking response, repeat the desktop lookup, and inspect the matching router logs:

```sh
/etc/init.d/dnsmasq restart
```

```powershell
nslookup -type=A -timeout=3 -retry=3 example.net. 192.0.2.1
```

```sh
logread -e dnsmasq | grep 'example.net' | tail -n 15
```

**Observed on 6 October 2026:** The user reported CT 100 stopped. The first request went to `.5`; approximately three seconds later a repeated query went to `1.1.1.1`. Public A records followed. Windows displayed one three-second timeout before the successful answer.

| Sequence | Observed event |
|---|---|
| First query | Query from `<LAB_CLIENT>` forwarded to AdGuard at `.5`. |
| About three seconds later | Repeated query forwarded to external fallback `1.1.1.1`. |
| Following reply | Public IPv4 answers returned. |

This establishes external fallback for the tested lookup with the AdGuard container stopped and Cudy's cache cleared. It does not establish zero-delay failover or identical behavior for every application. The observed fallback depends on the Windows client's retry after three seconds; strict-order is not a global health check that permanently switches all future requests to the fallback. New uncached queries may incur a delay while AdGuard remains unavailable. Filtering is bypassed during this public-fallback path.

The `udhcpc: no lease, failing` line also appears during dnsmasq restart. OpenWrt's startup script uses a DHCP-client probe to check for another DHCP server on a LAN interface, which is consistent with this message; the displayed message alone does not establish an ISP lease failure. The successful public-DNS exchange demonstrates upstream connectivity during the lookup.

References: [dnsmasq retry behavior](https://dnsmasq.org/docs/dnsmasq-man.html), [OpenWrt dnsmasq startup script and DHCP probe](https://git.openwrt.org/openwrt/staging/hauke/tree/?path=package/network/services/dnsmasq/files/dnsmasq.init).

### C: AdGuard restarted — passed after cache clearing

**Observed on 6 October 2026 local time:** The user reported CT 100 started again. After restarting dnsmasq to clear Cudy's cache, a desktop query from `<LAB_CLIENT>` was forwarded to `.5` and received the blocking answer `0.0.0.0`. The desktop lookup displayed that answer with no timeout message.

```text
query[A] example.net from <LAB_CLIENT>
forwarded example.net to 192.0.2.5
reply example.net is 0.0.0.0
```

| Scenario | Recorded result |
|---|---|
| AdGuard running | Passed: `.5` supplied the blocking answer. |
| AdGuard container stopped | Passed: public fallback `1.1.1.1` supplied IPv4 answers after one three-second client timeout. |
| AdGuard container restarted | Passed after router cache clearing: `.5` supplied the blocking answer again, with no displayed timeout. |

These tests validate the selected DNS paths for the tested desktop A-record lookup. Each scenario was run after a dnsmasq restart to clear router caches; recovery during an uninterrupted running dnsmasq instance and its existing caches was not separately tested. Cached public answers may persist until their TTL expires after AdGuard returns. No guarantee of instant failover or identical application retry behavior is implied.

### Test cleanup — instructions supplied, completion not yet recorded

1. In AdGuard's custom filtering rules, remove only the temporary `||example.net^` rule and apply the change.
2. On Cudy, disable diagnostic logging and restart dnsmasq to clear the cached blocking answer:

```sh
uci set 'dhcp.@dnsmasq[0].logqueries=0'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

3. From the desktop, confirm the test domain resolves to public addresses again with AdGuard still running:

```powershell
nslookup -type=A example.net. 192.0.2.1
```

Whole-Vaio shutdown and a live AdGuard instance with an unavailable Quad9 upstream have not been tested. Per-device default-DNS coverage has not been established for every wired/wireless device. Tailscale was pending at this stage; chapter 14 records its later deployment.

## 14 — Tailscale deployment on the VAIO

**User confirmed successful connection on 7 October 2026.** Tailscale runs in a separate native, unprivileged LXC on the VAIO, CT **102**, represented by `192.0.2.6`. It is not installed on Cudy. No account identity, approval URL, authentication key or VPN state is published.

Setup guidance included nesting/keyctl, TUN access at `/dev/net/tun`, installing curl and CA certificates, the official script, and service enablement:

```sh
apt update
apt install -y curl ca-certificates
curl -fsSL https://tailscale.com/install.sh | sh
systemctl enable --now tailscaled
```

Subnet-router guidance was:

```sh
# Historical example: substitute the real lab subnet before advertising.
cat > /etc/sysctl.d/99-tailscale.conf <<'EOF'
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF
sysctl -p /etc/sysctl.d/99-tailscale.conf
tailscale up --advertise-routes=192.0.2.0/24 --accept-dns=false
```

Authentication and route approval were discussed as part of setup, followed by the user's successful-connection report. Exact command outputs, the final route-approval view and a mobile-data end-to-end test were not supplied separately. Default subnet-router SNAT was retained in the proposed configuration.

`tailscaled` service enablement and Proxmox **Start at boot** are different: one starts the daemon inside the guest; the other starts the guest with its host. Both were advised, but final saved guest startup settings and a restart test were not recorded. Preserve guest VPN state through reboots. This access path is unavailable whenever the container or VAIO is off.

A later screenshot still showed a 1000 MiB cap. A 256–512 MiB starting cap was proposed; reduction is not demonstrated.

References: [Tailscale unprivileged LXC](https://tailscale.com/docs/features/containers/lxc/lxc-unprivileged), [subnet-router setup](https://tailscale.com/docs/features/subnet-routers/how-to/setup).

## 15 — HP Proxmox installation and updates

**Confirmed on 8 October 2026:** The HP's initial Windows 11 installation was replaced with Proxmox and web management was accessed. Its alias here is `pve-compute`, documentation address `192.0.2.4`.

| Installer observation | Recorded value |
|---|---|
| Target | Nominal 256 GB UMIS NVMe, about 238.47 GiB displayed |
| Filesystem | ext4 |
| Disk-size option | 238.0; other displayed size fields blank/automatic |
| Management interface | Wired `r8169` |
| Alternate interface | Wi-Fi `iwlwifi`, not selected for management |
| Address / gateway / DNS | `.4/24` / router `.1` / router `.1` |
| Pin interface names | Selected |

Full drive IDs, MAC addresses and original hostname are omitted. SVM CPU virtualization was enabled in BIOS. AMD DASH was also selected; disabling it if unused was advised, but its final state is unverified. The final FQDN suffix was not recorded. An initial 401 login error was followed by successful login; its cause was not established.

Initially HP reported PVE 9.2.2 with kernel `7.0.2-6-pve`; VAIO reported PVE 9.2.21 with kernel `7.0.14-20-pve`. After updates, both reported:

```text
pve-manager/9.2.21/4f6e0ac86f9e8c7f
running kernel: 7.0.14-22-pve
```

HP `free -h` showed 6.7 GiB usable RAM, 1.6 GiB used, 5.1 GiB available and 6.7 GiB swap with none used. Unit conversion and hardware reservations can contribute to the difference from 8 GB installed. Exact GPU-reserved memory was not measured. Final partition sizes were not independently listed.

## 16 — Proxmox Datacenter Manager

**Confirmed on 8 October 2026:** PDM **1.1.7** runs on the VAIO and displays both independent Proxmox hosts online. A PVE cluster was not created; there is no shared quorum dependency when the VAIO is shut down.

| Guest setting | Recorded configuration |
|---|---|
| CT / host | 200 / `pve-core` |
| OS | Debian 13 |
| Privilege | Unprivileged |
| CPU | One-core limit |
| RAM / swap | 1 GiB / 512 MiB cap |
| Storage | 16 GiB allocated on `local-lvm` |
| Address / interface | Fictional `.10` / HTTPS port 8443 |

Native installation used `proxmox-datacenter-manager-container-meta`. Guidance covered Debian 13/Trixie and `pdm-no-subscription`; a complete final repository listing was not captured. This was not a Docker installation.

Two remotes were added, anonymized as `core` and `compute`. A remote ID is a local management label, not a username, IP or cluster-join operation. The dashboard showed two reachable remotes, two online PVE nodes, no VMs, three running containers and one stopped container. AdGuard, Tailscale and PDM were the running guests; the stopped guest's role was unrecorded.

No PBS was registered in that view; this does not establish whether separate PVE backup jobs exist. A no-valid-subscription notice does not indicate failed connectivity. Start at boot was advised, but the saved setting and a host-reboot test were not recorded. Hosts continue operating when the PDM dashboard is down. PDM does not pool RAM/storage, replicate guests or provide HA.

References: [PDM installation](https://pdm.proxmox.com/docs/installation.html), [remotes](https://pdm.proxmox.com/docs/remotes.html), [web UI](https://pdm.proxmox.com/docs/web-ui.html), [subscription FAQ](https://pdm.proxmox.com/docs/faq.html).

## 17 — Memory review and service planning

The latest VAIO snapshot with PDM running showed **3.8 GiB total, 1.8 GiB used, 2.0 GiB available**, and **32 KiB swap used**. AdGuard used **61.35 MiB / 1000 MiB**, Tailscale **32.11 MiB / 1000 MiB**, and PDM **110.06 MiB / 1 GiB**. PDM disk use was **2.52 GiB / 15.58 GiB**. These are light-activity snapshots, not peak-load measurements.

LXC memory caps limit growth; they do not reserve that entire amount as physical RAM. Reducing an idle cap does not release the difference as previously reserved memory. Linux `available` includes reclaimable memory; `free` alone understates usable headroom. PDM's small observed footprint does not establish its peak need, so retaining its 1 GiB cap was proposed.

**Nginx Proxy Manager remains planned**, represented by `.7`. A tentative VAIO guest budget was one core, 768 MiB RAM, 256 MiB swap and 8 GiB disk, using SQLite. No guest, installation or peak-load test is demonstrated. NPM is a poor fit for Cudy's limited persistent flash; plain Nginx is smaller, and the conclusion does not apply to every possible Nginx configuration.

A password manager was mentioned for later, with Bitwarden/Vaultwarden options; neither is installed. Media, file-sync, Homepage, Uptime Kuma and other compute services remain possible future work. Hardware transcoding has not been configured or tested. PDM aggregate figures are totals across independent nodes, not a memory/storage pool.

Earlier guidance that nesting never matters for native services was too broad: modern Debian/systemd workloads can need it for isolation features even without Docker. No nesting reduction is recorded as performed.

References: [PVE LXC memory settings](https://github.com/proxmox/pve-docs/blob/master/pct.adoc), [Debian free manual](https://manpages.debian.org/trixie/procps/free.1.en.html), [NPM setup](https://nginxproxymanager.com/setup/), [systemd/nesting guidance](https://github.com/proxmox/pve-docs/blob/master/pct.adoc).

## 18 — LAN port scan; firewall review deferred

**Observed on 8 October 2026:** User-supplied LAN TCP results are recorded in [sanitized evidence](sanitized-evidence.md). The invocation was not shown. Counts are consistent with a common default 1000-port scan, but exact options and coverage remain unverified. UDP and all 65,535 TCP ports were not demonstrated as tested.

Open listeners reachable from the LAN do not prove public internet exposure. Basic service labels identify conventional port names, not confirmed installed software or versions. `squid-http` on 3128 does not prove Squid is installed; Proxmox commonly uses it for the SPICE proxy. HP's TCP 111 listener needs process identification. Port 8006 did not appear in the displayed scan even though PVE web access works; omission is not proof it is closed.

No current WAN rules, upstream forwarding/DMZ/UPnP inventory, public IPv6 audit or external-source test was supplied. Internet exposure is therefore **unverified**. A reverse proxy does not close backend ports or replace firewall rules. Same-subnet traffic normally travels through the switch without crossing the router's routed firewall.

Proposed next read-only checks were `uci show firewall` on OpenWrt and `ss -lntup` on HP. The user paused before that review. No resulting outputs, port closures, new firewall rules or NPM installation are recorded. Future restrictions must preserve intended PDM-to-PVE access on 8006 and Tailscale management paths.

The session ended with documentation and privacy cleanup only. Further configuration is deferred.

References: [Nmap port states](https://nmap.org/book/man-port-scanning-basics.html), [port selection](https://nmap.org/book/man-port-specification.html), [PVE firewall/service ports](https://github.com/proxmox/pve-docs/blob/master/pve-firewall.adoc), [OpenWrt security](https://openwrt.org/docs/guide-user/security/openwrt_security).


## 19 — Guest Wi-Fi and subnet separation

**User confirmed working internet and isolated Guest networking on 11 October.** The Guest AP is mapped to `guest`, using `br-guest` with no physical switch member. Its gateway is represented here by `203.0.113.1/24`.

| Setting | Recorded / supplied value |
|---|---|
| Guest DHCP | IPv4 enabled; start 100, limit 150, lease 12h |
| Client DNS advertisement | DHCP option `6,203.0.113.1` |
| Zone | INPUT Reject; OUTPUT Accept; forwarding to WAN only |
| Router DNS exception | TCP/UDP destination 53 on own Guest gateway |
| Router DHCP exception | UDP source 68 → destination 67; no fixed destination IP |
| Upstream private-network block | Guest → WAN destination `198.51.100.0/24`, Reject, IPv4 |
| Wireless peer isolation | Supplied; final two-client validation not recorded |

DHCP option 6 advertises a DNS-server address, not a port. DNS/DHCP exceptions accept traffic into Cudy itself. Cudy then forwards DNS to AdGuard, using the existing upstream/fallback design. No direct Guest access to AdGuard's dashboard is intended.

In this LuCI workflow, the destination selector did not expose the intended router-input choice. Cloning a working INPUT rule preserved the needed classification. A rule with a destination forwarding zone is a different path. Completed rules were checked for **Accept input**.

Guest isolation is based on firewall policy as well as a separate subnet. A hidden SSID is not an access-control boundary. A captive portal, open SSID and 24-hour access expiry from an example article were not deployed. IPv6/peer isolation still need full tests.

## 20 — Switch discovery and management recovery

The selected static switch endpoint became unreachable during troubleshooting. An enabled desktop VPN was noticed and disabled. Several temporary desktop addressing/ARP checks followed; an absent ARP-cache entry alone was not sufficient to declare the switch offline.

The **Omada Discovery Utility** found the ES205G at a different DHCP address within the existing LAN. The owner reapplied the selected static address, represented by `192.0.2.2`, and confirmed access.

The restoration guidance included mask `/24`, gateway `192.0.2.1`, disabled switch DHCP client and separate startup **Save**. The final gateway and reboot persistence were not independently verified. Reported power outages do not prove that configuration loss caused the address change.

Windows command-shell differences were clarified: `Get-NetAdapterBinding` is a PowerShell cmdlet, while `netsh` addressing commands can be used from CMD. Desktop automatic IP/DNS restoration was instructed. Account names, MACs and original temporary addresses are omitted.

## 21 — Final VLAN IDs and physical-port correction

The owner's latest selection is **VLAN 10 Homelab** and **VLAN 20 Trusted**. Guest remains Wi-Fi-only on its own bridge/subnet. This supersedes the early drawing with Guest VLAN 10 and Homelab VLAN 30.

The latest physical drawing also establishes **switch port 2 → VAIO** and **port 3 → HP**, replacing an earlier reversed plan. Port 1 goes to Cudy LAN1, port 4 is spare and port 5 goes to the desktop.

The switch was shown with 802.1Q enabled, all ports untagged in VLAN 1/PVID 1, and port 1 tagged in VLANs 10 and 20. Other ports were nonmembers of the new VLANs. Management remained VLAN 1 and ingress checking was enabled.

This prepares the trunk definitions without moving endpoint traffic. Startup persistence remains untested.

## 22 — Cudy bridge VLAN and Trusted interface staging

The supplied bridge table retained VLAN 1 untagged/PVID on LAN1–3, added tagged VLANs 10/20 on LAN1 only, and checked Local participation. The final complete table was not separately returned.

A later interface screenshot showed:

| Interface | Device | State |
|---|---|---|
| `guest` | `br-guest` | Separate Guest gateway, active |
| `lan` | `br-lan.1` | Existing LAN gateway, active |
| `trusted` | `br-lan.20` | Separate Trusted gateway, active; no client test recorded |
| Extra `homelab` | `br-lan.10` | Marked for deletion |

The chosen migration is to **reuse `lan` for Homelab** and later change only its device to `br-lan.10`, keeping server/service addresses. Do not add a second static interface for the same LAN subnet. Apply instructions were supplied for deleting the extra `homelab`; the user proceeded, but final absence is not shown. The VLAN 10 bridge definition must remain.

Trusted DHCP guidance was start 100, limit 150, lease 12h and option `6,<TRUSTED_GW>`, with DHCPv4 enabled and RA/DHCPv6/NDP disabled for the initial IPv4 stage. Those final DHCP settings were not fully audited. The main AP still belongs to `lan` in the last verified mapping.

Wired WAN carrier absence is expected when the actual uplink is wireless `wwan`; it is not by itself an ISP outage.

## 23 — Trusted firewall exceptions

The Trusted zone screenshot showed INPUT Reject, OUTPUT Accept and intra-zone forwarding Reject. WAN-only forwarding was instructed, with no broad Trusted → LAN forwarding. Final saved forwarding is pending inspection.

Three reviewed INPUT rules cover router DNS, DHCP and administration. They had both address families selected. Corrections were supplied to use IPv4 only and explicitly select TCP + UDP for DNS. The screenshot did not establish the exact DNS protocol selection.

Administration FORWARD rules were then supplied for Trusted → existing `lan`: PVE hosts `.3`/`.4` on TCP 22/8006, PDM `.10` on TCP 8443, and switch `.2` on TCP 80. Final rule summaries and live Trusted tests were not returned.

The rule names and fields are listed in the [migration runbook](vlan-migration.md). Stateful replies do not require a broad reciprocal initiation rule. Same-VLAN traffic bypasses the router firewall.

## 24 — AdGuard dashboard port and latest stopping point

The user first said the bare AdGuard address and `:3000` gave the same result, then clarified that adding `:3000` caused trouble. That clarification did not itself provide a final browser URL, listener listing or successful HTTP response transcript.

Port 80 appeared in the earlier scan. The latest instruction was to use the working bare address and inspect its final scheme: **HTTP means TCP 80; HTTPS without an explicit port means TCP 443**. A port-specific redirect/listener check remains available if needed.

A conditional `Allow-Trusted-AdGuard-Admin` rule was supplied: source `trusted`, destination `lan`, AdGuard `.5`, TCP on the verified dashboard port, Accept, IPv4 only and blank source IP/port. It should say **Accept forward**. No saved summary was returned before the documentation request. Do not declare port 3000 allowed or the Trusted dashboard path tested.

## 25 — Migration still pending

Next stages are rule/interface verification; one Trusted wireless test device while desktop remains wired on VLAN 1; desktop port 5 migration to VLAN 20; explicit switch management decision; then Homelab server/router cutover to VLAN 10.

Keeping switch management in Homelab with its current address preserves the baseline but permits same-VLAN server access. Moving it to Trusted requires a changed IP/gateway and revised rule. Neither choice has been finalized.

The final migration should preserve current Proxmox/guest IPs, PDM remotes and DNS upstreams. Hosts/guests remain untagged behind switch access ports. Post-cutover isolation, DNS fallback, remote Tailscale, restart persistence and IPv6 tests remain outstanding.

## 26 — Documentation and handoff update

On 11 October the user requested updated documentation, a handoff and planned future goals. The package now includes current/target diagrams, rule/status ledgers, migration gates, a self-contained handoff and an ordered roadmap.

The immediate priority is completing network separation. Backups/restore and availability tests follow; NPM on VAIO and a single useful HP workload are later goals. Media, file sync, password management and an AI-security sandbox remain options, not deployed services.

Real network/device/account identifiers and source images remain excluded. This update did not change equipment settings or publish a GitHub repository.

References for chapters 19–26: [OpenWrt firewall](https://openwrt.org/docs/guide-user/firewall/firewall_configuration), [DSA](https://openwrt.org/docs/guide-user/network/dsa/converting-to-dsa), [DHCP](https://openwrt.org/docs/guide-user/base-system/dhcp), [ES205G user guide](https://static.tp-link.com/upload/manual/2024/202408/20240813/1910013715_Omada%20Easy%20Managed%20Switch_UG.pdf), [AdGuard configuration](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration), [PDM web UI](https://pdm.proxmox.com/docs/web-ui.html).
