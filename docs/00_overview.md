# 00_overview.md

## Homelab DNS + Reverse Proxy Server

### Purpose
This document provides a technical overview of the local DNS and Reverse Proxy server used in the home network.  
It acts as the central gateway for **name resolution, ad-blocking, and internal service routing**.

The goal of this setup is to:
- Improve **network performance** and **privacy** with local recursive DNS.  
- Centralize **service access** via clean, user-friendly hostnames (e.g. `jellyfin.home.lan`).  
- Provide a single, maintainable point of configuration for all internal HTTP services.  

---

## System Overview

| Component | Role | Description |
|------------|------|-------------|
| **Pi-hole** | DNS Forwarder | Caches local DNS requests and blocks ads across the network. |
| **Unbound** | Recursive Resolver | Handles secure DNS recursion and DNSSEC validation. |
| **Nginx** | Reverse Proxy | Routes web requests for internal services by hostname. |
| **OpenWRT Router** | DHCP / Upstream DNS | Assigns IPs and directs DNS queries to the proxy server. |

---


## Network Summary

| Hostname | IP Address | Services | Purpose |
|-----------|-------------|----------|----------|
| `dnsproxy.home.lan` | `192.168.1.x` | Pi-hole, Unbound, Nginx | DNS + Reverse Proxy |
| `media.home.lan` | `192.168.1.x` | Jellyfin, ARM | Media Server |
| `openwrt.lan` | `192.168.1.1` | DHCP, Gateway | Network Router |

---

## Key Features

### DNS Layer
- **Ad-blocking:** via Pi-hole’s blocklists.  
- **Recursive resolution:** using Unbound with DNSSEC validation.  
- **Caching:** dramatically reduces lookup latency.  
- **Local hostname resolution:** integrates with OpenWRT DHCP for `.home.lan` domains.  

### Proxy Layer
- **Central routing:** Nginx handles all service domains on port 80.  
- **Simplified URLs:** access internal services via names instead of ports.  
- **Isolation:** real service ports (8080, 8096, etc.) are hidden behind the proxy.  

---

## System Behavior

| Action | Result |
|--------|---------|
| Client sends a DNS query for `service.home.lan` | Pi-hole responds with `192.168.1.x (DNS Server IP)` |
| Client connects to `http://service.home.lan` | Nginx proxies the request to `192.168.1.x:port_number of server it's running on` |
| External DNS request (e.g. `google.com`) | Unbound securely resolves the query via root DNS servers |
| OpenWRT renews DHCP leases | Pi-hole updates host records dynamically |

---

## Related Documents

| File                                                     | Description                                |
| -------------------------------------------------------- | ------------------------------------------ |
| [`01_dns_unbound_setup.md`](01_dns_unbound_setup.md)     | Unbound installation and DNSSEC validation |
| [`02_pihole_integration.md`](02_pihole_integration.md)   | Pi-hole + Unbound configuration            |
| [`03_nginx_reverse_proxy.md`](03_nginx_reverse_proxy.md) | Nginx virtual host and proxy setup         |
| [`04_troubleshooting.md`](04_troubleshooting.md)         | Recovery and debugging guide               |
