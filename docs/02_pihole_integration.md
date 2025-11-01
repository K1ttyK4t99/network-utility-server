# 02_pihole_integration.md
  
## Pi-hole Integration with Unbound and Network DNS

### Purpose
This document explains how to configure **Pi-hole** as a local network-wide ad blocker and DNS forwarder, integrated with **Unbound** for secure recursive resolution.

The goal is to centralize DNS queries on the Pi-hole server (`192.168.1.x`) so all devices benefit from:
- Local caching for speed  
- DNSSEC validation via Unbound  
- Ad and tracker blocking  
- Internal hostname resolution (e.g., `*.home.lan`)  
- Seamless integration with OpenWRT DHCP assignments

---

## System Overview

| Component | Role |
|------------|------|
| **Unbound** | Validating recursive resolver running locally on port `5335` |
| **Pi-hole** | Network DNS forwarder, ad blocker, and hostname manager |
| **OpenWRT** | DHCP server, assigns Pi-hole as DNS server to clients |

---

## Installation Steps

### 1. Install Pi-hole
Run the official installation command:

```bash
curl -sSL https://install.pi-hole.net | bash
```

During setup:
- **Select "Static IP"** -> confirm your Pi-hole server IP (192.168.1.x)
- **Upstream DNS** -> Choose **Custom** and enter:
```bash
127.0.0.1#5335
```
- **Blocklists** -> accept defaults (these can be customized later)
- **Web Admin Interface** -> yes
- **Web Server** -> yes
- **Log queries** -> optional (recommended: "Yes, show everything")

After setup, note the admin password shown on screen (you can change it later via):
```bash
pihole -a -p
```

---

## Integration with Unbound

### 2. Configure Pi-hole to use Unbound
Edit Pi-hole's upstream configuration as shown in [01_dns_unbound_setup](01_dns_unbound_setup)

---

## Network Integration via OpenWRT

### 3. Set Pi-hole as the DNS Server for All Devices 
On OpenWRT:
  1. Navigate to **Network** -> **Interfaces** -> **LAN** -> **DHCP Server** -> **Advanced Settings**
  2. Under **DHCP-Options**, add:
  ```
  6,192.168.1.x
  ```
  (this pushes Pi-hole's IP as the DNS server via DHCP option 6.)
  3. Save & Apply
  
---

## Useful Commands

| Command                              | Description                                  |
| ------------------------------------ | -------------------------------------------- |
| `pihole -g`                          | Update gravity lists (ad/tracker blocklists) |
| `pihole -a -p`                       | Change web admin password                    |
| `sudp systemctl restart pihole-FTL`  | Restart Pi-hole DNS service                  |
| `sudo systemctl status pihole-FTL`   | Check Pi-hole DNS service status             |
| `pihole -t`                          | View live DNS query log                      |
| `dig @127.0.0.1 -p 5335 example.com` | Test Unbound directly                        |

---

## Maintenance

| Task                      | Frequency | Command                             |
| ------------------------- | --------- | ----------------------------------- |
| Update Pi-hole            | Monthly   | `pihole -up`                        |
| Update blocklists         | Weekly    | `pihole -g`                         |
| Restart Pi-hole services  | As needed | `sudo systemctl restart pihole-FTL` |
| Verify Unbound connection | Monthly   | `dig @127.0.0.1 -p 5335 google.com` |

