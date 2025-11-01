# 07_troubleshooting.md 

## Troubleshooting Guide  

This document covers common problems, diagnostic commands, and recovery steps for the DNS and reverse proxy stack.

---

## General Approach

Before making changes:
1. Verify **system health**:
   ```bash
   uptime
   df -h
   free -h
   ```
2. Check network connectivity:
   ```bash
   ping 8.8.8.8
   ping service.home.lan
   nslookup service.home.lan
   ```
   
---

## DNS (Unbound + Pi-hole)

### DNS Resolution Fails
**Symptoms:**
- No Internet access
- `nslookup` or `dig` returns "SERVFAIL" or "NXDOMAIN"

**Checks:**
```bash
systemctl status unbound
journalctl -u unbound --no-pager | tail -n 20
sudo unbound-checkconf
```

**Fixes:**
- Ensure Unbound is listening on `127.0.0.1#5335`
- Check Pi-hole settings to confirm upstream points to `127.0.0.1#5335`
- Restart both services:
 ```bash
 systemctl restart Unbound
 systemctl restart pihole-FTL
 ```
 
---

## Reverse Proxy (Nginx)
**Symptoms:**
Site returns 502 Bad Gateway

**Checks:**
```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx --no-pager | tail -n 20
```

**Fixes:**
- Make sure the backend service is up and reachable:
```bash
curl -v http://<ip address>:<backend_port>
```
- Reload Nginx
```bash
sudo systemctl restart Nginx
```

---

## Diagnostic Tools

| Tool         | Purpose                   | Example Command              |
| ------------ | ------------------------- | ---------------------------- |
| `dig`        | DNS record query          | `dig @127.0.0.1 example.com` |
| `nslookup`   | Simple DNS test           | `nslookup service.home.lan`  |
| `ss`         | Check open ports          | `ss -tuln`                   |
| `curl`       | Test HTTP/HTTPS endpoints | `curl -v http://<ip address>`|
| `journalctl` | Check service logs        | `journalctl -u nginx -n 50`  |

---

## Common Fix-All Steps 
When in doubt:
```bash
sudo systemctl restart unbound
sudo systemctl restart pihole-FTL
sudo systemctl restart nginx
```
Then re-run:
```bash
ping 8.8.8.8
nslookup example.com
curl -v http://service.home.lan
```

---

**Tip:** Keep this system minimal. Every added package increases potential attack surface and complexity. For long-term reliability, schedule weekly health checks using cron and email logs or summaries to yourself.
