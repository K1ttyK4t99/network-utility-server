# Homelab Infrastructure Documentation

## Overview
This repository documents the setup, maintenance, and recovery procedures for a self-hosted **DNS and Reverse Proxy Server**.  
It contains **documentation and scripts only** — no live configuration files or sensitive data are stored here.

The system currently runs:
- **Pi-hole** – network-wide ad blocking and DNS relay  
- **Unbound** – recursive DNS resolver for privacy and performance  
- **Nginx** – reverse proxy for internal web services  
- **OpenWRT Router** – DHCP and upstream DNS management  

---

## Network Summary

| Role | Hostname | IP Address | Description |
|------|-----------|-------------|-------------|
| Reverse Proxy / DNS | `dnsproxy.home.lan` | `192.168.1.x` | Nginx, Pi-hole, Unbound |
| Media Server | `media.home.lan` | `192.168.1.x` | Jellyfin, Nextcloud |
| Router | `openwrt.lan` | `192.168.1.1` | DHCP, Firewall, Gateway |

**Documentaion:**
- [docs/00_overview.md](docs/00_overview.md)



---

## Service Summary

### Pi-hole + Unbound
- Acts as the primary DNS for the network.
- Forwards all recursive queries securely via Unbound.
- Integrated with OpenWRT DHCP for automatic client DNS resolution.

**Documentation:**  
- [`docs/01_dns_unbound_setup.md`](docs/01_dns_unbound_setup.md)  
- [`docs/02_pihole_integration.md`](docs/02_pihole_integration.md)

---

### Nginx Reverse Proxy
- Provides clean URLs for internal services (e.g. `example.home.lan`).  
- Handles HTTP forwarding and optional SSL termination.  
- Routes requests based on `server_name`.

**Example Configuration:**  
See [`examples/sample_nginx_reverse_proxy.conf`](examples/sample_nginx_reverse_proxy.conf)

---


## Troubleshooting

If a service stops responding:

1. Check system service status:
   ```bash
   systemctl status pihole-FTL unbound nginx
   ```
2. Verify DNS resolution:
   ```bash
   nslookup service.home.lan 192.168.1.x (IP address of server)
   ```
3. Test proxy connectivity:
   ```bash
   curl -v http://service.home.lan
   ```
4. If DNS fails, restart Pi-hole and Unbound:
   ```bash
   sudo systemctl restart pihole-FTL Unbound
   ```
Full guide: [docs/04_troubleshooting.md](docs/04_troubleshooting.md)

## Future Improvements
- Add SSL certificates for all internal domains
- Implement automatic health checks and alerting