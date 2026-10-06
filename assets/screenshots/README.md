# Configuration screenshots

Place the selected progress photographs in this directory. AdGuard creation, direct DNS tests and DNS prerequisite screenshots are embedded in the journal.

## Suggested filenames

| Stage | Example filename |
|---|---|
| Hardware identification | `01-cudy-hardware-label.jpg` |
| OpenWrt result | `02-openwrt-status.png` |
| LAN and DHCP | `03-lan-dhcp-settings.png` |
| Wireless ISP uplink | `04-wireless-uplink.png` |
| Local Wi-Fi | `04-lan-access-point.png` |
| Storage and package inspection | `05-router-storage.png`, `05-vpn-simulation.png` |
| Switch management | `06-switch-static-ip.png`, `06-switch-status.png` |
| Desktop and cabling | `07-physical-connections.jpg`, `07-desktop-ipconfig.png` |
| HP installation | `08-hp-installation.png` |
| Vaio configuration | `08-vaio-configuration.png` |

Use the file's actual extension, simple filenames without spaces, and one readable photograph per documented result. Keep the original photos outside the public repository if you create edited copies.

Before publishing, obscure visible passwords, Wi-Fi keys, VPN/authentication tokens and private keys. Include model and hardware revision in captions without needing to publish full serial numbers.

Embed images using paths relative to the Markdown document. In the journal, for example:

```markdown
![OpenWrt version and router model](../assets/screenshots/02-openwrt-status.png)
```

For each image, record what it proves. An upload screen alone does not prove a successful installation; use the subsequent running-system status page as well.

## Added evidence

The five `09-01` through `09-05` AdGuard creation-wizard screenshots have been added as supplied, without image edits:

- `09-01-adguard-general.png`: General: CT 100, hostname AdGuard, unprivileged and nesting selected.
- `09-02-adguard-disk.png`: Disk: 5 GiB on local-lvm.
- `09-03-adguard-memory.png`: Memory: 1000 MiB RAM and 512 MiB swap limit.
- `09-04-adguard-network.png`: Network: vmbr0, static 192.168.2.5/24, gateway 192.168.2.1, no VLAN tag.
- `09-05-adguard-dns.png`: DNS: host settings inherited in the wizard.

These show configuration in progress, rather than a completed deployment.

- `10-01-adguard-configure-devices.png`: AdGuard setup instructions and listener address.
- `10-02-adguard-dns-tests.png`: Successful direct DNS lookups from the desktop.

- `11-01-adguard-upstream.png`: AdGuard public DNS-over-HTTPS upstream.
- `11-02-router-dns-preflight.png`: OpenWrt configuration, dnsmasq version and direct public-DNS test.
- `11-03-public-dns-test.png`: Completed successful direct lookup against `1.1.1.1`.

- `12-01-router-adguard-query-log.png`: Recent queries around 19:15 with Cudy at `192.168.2.1` shown as the client.

- `13-01-adguard-running-lookup.png`: Desktop lookup through Cudy returns `0.0.0.0` for the temporarily blocked test domain.
- `13-02-adguard-running-router-log.png`: Cudy forwards that query to `.5`, receives the blocking answer and caches a subsequent response.

- `13-03-adguard-stopped-router-log.png`: AdGuard attempt followed by forwarding to `1.1.1.1` on a client retry three seconds later.
- `13-04-adguard-stopped-lookup.png`: Public IPv4 answers after one three-second timeout with CT 100 reported stopped.

- `13-05-adguard-restored-router-log.png`: Query at 19:40:55 forwarded to `.5` with the blocking answer after CT 100 restarted and Cudy cache clearing.
- `13-06-adguard-restored-lookup.png`: Desktop receives `0.0.0.0` again, with no timeout displayed.
