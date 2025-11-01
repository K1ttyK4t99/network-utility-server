# 01_dns_unbound_setup.md
  
## Local Recursive DNS Resolver with Unbound

### Purpose
This document describes the installation and configuration of **Unbound**, a validating, recursive, and caching DNS resolver.  
Unbound is used by the Pi-hole server (`192.168.1.x`) to provide secure, local DNS recursion **without relying on external DNS providers**.

By combining Unbound with Pi-hole, the network achieves:
- **Full DNS independence** (no reliance on Google, Cloudflare, or ISPs).  
- **DNSSEC validation** for authenticity and integrity.  
- **Improved privacy** with direct queries to root DNS servers.  
- **Faster, cached responses** for frequently visited domains.

---

## System Overview

| Component | Role |
|------------|------|
| **Unbound** | Recursive DNS resolver, running locally on port `5335`. |
| **Pi-hole** | Acts as the network DNS forwarder; sends queries to Unbound. |
| **OpenWRT** | Assigns clients to use Pi-hole (`192.168.1.x`) as their DNS server. |

---

## Installation Steps

### 1. Install Unbound
On the DNS server (`192.168.1.164`):

```bash
sudo apt update
sudo apt install unbound -y
```

### 2. Verify the installed version
Check the version to confirm DNSSEC-capable build:

```bash
unbound -v
```
Expected output should include:

```bash
with DNSSEC support
```

---

## Configuration

### Create a custom configuration file
Backup the default configuration and create a local one:

```bash
sudo mv /etc/unbound/unbound.conf /etc/unbound/unbound.conf.bak
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

Paste the following configuration:

```conf
server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no
    root-hints: "/var/lib/unbound/root.hints"
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no
    edns-buffer-size: 1232
    prefetch: yes
    qname-minimisation: yes
    cache-min-ttl: 3600
    hide-identity: yes
    hide-version: yes
    unwanted-reply-threshold: 10000
    val-clean-additional: yes
    so-rcvbuf: 1m
    so-sndbuf: 1m
    auto-trust-anchor-file: "/var/lib/unbound/root.key"

forward-zone:
    name: "."
    forward-addr: 9.9.9.9@853#dns.quad9.net
    forward-addr: 149.112.112.112@853#dns.quad9.net
    forward-tls-upstream: yes
```

---

## Root hints
Download the latest root hints file:

```bash
sudo curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```

You can refresh this monthly using a cron job:

```bash
sudo crontab -e
```

Add:

```conf
@monthly curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```

---

## DNSSEC Trust Anchor
Initialize the DNSSEC root key

```bash
sudo unbound-anchor -a /var/lib/unbound/root.key
```

If unbound-anchor is installed, install it

```bash
sudo apt install unbound-anchor
```

---

## Start and Enable Unbound

```bash
sudo systemctl enable unbound
sudo systemctl start unbound
sudo systemctl status unbound
```

---

## Testing
**Check Local Resolution**

Query Unbound directly:

```bash
dig @127.0.0.1 -p 5335 example.com
```

Expected output:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, ...
;; SERVER: 127.0.0.1#5335(127.0.0.1)

**Check DNSSEC validation**
Run:
```bash
dig @127.0.0.1 -p 5335 sigfail.verteiltesysteme.net
dig @127.0.0.1 -p 5335 sigok.verteiltesysteme.net
```

You should see:
- sigfail.verteilesysteme.net -> SERVFAIL
- sigok.verteilesysteme.net -> NOERROR

If so, DNSSEC is functioning correctly

---

## Integration with Pi-hole
In the Pi-hole admin interface:
   1. Go to Settings -> DNS
   2. Disable all upstream DNS servers
   3. Enable Custom 1 (IPv4) and set to:
   ```bash
   127.0.0.1#5335
   ```
   4. Save and restart Pi-hole:
   ```bash
   sudo systemctl restart pihole-FTL
   ```
You can confirm Pi-Hole is using Unbound:
```bash
dig google.com @127.0.0.1 -p 5335
```

---

## Verification Steps

| Test             | Command                                                | Expected Result       |
| ---------------- | ------------------------------------------------------ | --------------------- |
| Unbound status   | `sudo systemctl status unbound`                        | Active (running)      |
| DNSSEC test      | `dig sigok.verteiltesysteme.net`                       | `NOERROR`             |
| External query   | `dig google.com`                                       | Resolves successfully |

## Maintenance

| Task               | Frequency | Command                                                                               |
| ------------------ | --------- | ------------------------------------------------------------------------------------- |
| Refresh root hints | Monthly   | `sudo curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root` |
| Restart service    | As needed | `sudo systemctl restart unbound`                                                      |
| Update packages    | Weekly    | `sudo apt update && sudo apt upgrade -y`                                              |



