# Configuration screenshots

Place the selected progress photographs in this directory. AdGuard creation, direct DNS tests and DNS prerequisite screenshots are embedded in the journal.

## Suggested filenames

| Stage | Example filename |
|---|---|
| Hardware identification | `cudy-hardware-label.jpg` |
| OpenWrt result | `openwrt-status.png` |
| LAN and DHCP | `lan-dhcp-settings.png` |
| Wireless ISP uplink | `wireless-uplink.png` |
| Local Wi-Fi | `lan-access-point.png` |
| Storage and package inspection | `router-storage.png`, `vpn-simulation.png` |
| Switch management | `switch-static-ip.png`, `switch-status.png` |
| Desktop and cabling | `physical-connections.jpg`, `desktop-ipconfig.png` |
| HP installation | `hp-installation.png` |
| Vaio configuration | `vaio-configuration.png` |

Use the file's actual extension, simple filenames without spaces, and one readable photograph per documented result. Keep the original photos outside the public repository if you create edited copies.

Before publishing, obscure visible passwords, Wi-Fi keys, VPN/authentication tokens and private keys. Include model and hardware revision in captions without needing to publish full serial numbers.

Embed images using paths relative to the Markdown document. In the journal, for example:

```markdown
![OpenWrt version and router model](../assets/screenshots/openwrt-status.png)
```

For each image, record what it proves. An upload screen alone does not prove a successful installation; use the subsequent running-system status page as well.

## Added evidence

- `adguard-general.png`: General: CT 100, hostname AdGuard, unprivileged and nesting selected.
- `adguard-disk.png`: Disk: 5 GiB on local-lvm.
- `adguard-memory.png`: Memory: 1000 MiB RAM and 512 MiB swap limit.
- `adguard-network.png`: Network: vmbr0, static 192.168.2.5/24, gateway 192.168.2.1, no VLAN tag.
- `adguard-dns.png`: DNS: host settings inherited in the wizard.

These show configuration in progress, rather than a completed deployment.

- `adguard-configure-devices.png`: AdGuard setup instructions and listener address.
- `adguard-dns-tests.png`: Successful direct DNS lookups from the desktop.

- `adguard-upstream.png`: AdGuard public DNS-over-HTTPS upstream.
- `router-dns-preflight.png`: OpenWrt configuration, dnsmasq version and direct public-DNS test.
- `public-dns-test.png`: Completed successful direct lookup against `1.1.1.1`.

- `router-adguard-query-log.png`: Recent queries around 19:15 with Cudy at `192.168.2.1` shown as the client.

- `adguard-running-lookup.png`: Desktop lookup through Cudy returns `0.0.0.0` for the temporarily blocked test domain.
- `adguard-running-router-log.png`: Cudy forwards that query to `.5`, receives the blocking answer and caches a subsequent response.

- `adguard-stopped-router-log.png`: AdGuard attempt followed by forwarding to `1.1.1.1` on a client retry three seconds later.
- `adguard-stopped-lookup.png`: Public IPv4 answers after one three-second timeout with CT 100 reported stopped.

- `adguard-restored-router-log.png`: Query at 19:40:55 forwarded to `.5` with the blocking answer after CT 100 restarted and Cudy cache clearing.
- `adguard-restored-lookup.png`: Desktop receives `0.0.0.0` again, with no timeout displayed.
