# 03_nginx_reverse_proxy.md  

## Reverse Proxy and Local Service Integration (NGINX)

### Purpose
This document explains how to configure **NGINX** as a local reverse proxy for internal web services (like Jellyfin or others).  

Using a reverse proxy allows access to services through friendly hostnames such as:
http://jellyfin.home.lan
instead of raw IP addresses or port numbers
This is useful if you have multiple services running on the same server, as you cannot map ports to DNS records, but you can do so with a reverse proxy.

---


## System Overview

| Component | Role |
|------------|------|
| **NGINX (192.168.1.x)** | Central reverse proxy for local web services |
| **Pi-hole (192.168.1.x)** | Local DNS server resolving `*.home.lan` domains |
| **Service Hosts** | Internal servers running applications (e.g., Jellyfin at `192.168.1.143:8096`) |
| **OpenWRT Router** | Provides DHCP and assigns Pi-hole as DNS for all LAN clients |

---

## 1. Install NGINX

### Debian / Ubuntu:
```bash
sudo apt update
sudo apt install nginx -y
```
Confirm it's running:
```bash
sudo systemctl status nginx
```

---

## 2. Directory Layout
Store per-service proxy configs in a dedicated directory structure

```
/etc/nginx/
├── sites-available/
│   ├── example1
│   ├── example2
│   └── default
└── sites-enabled/
    ├── example1 -> ../sites-available/example1
    ├── example2 -> ../sites-available/example2
```

---

## 3. Create Reverse Proxy Configs 

### Example:
```nginx
server {
    listen 80;
    server_name example.home.lan;

    location / {
        proxy_pass http://192.168.1.x:port-number; # This is the IP address and port number of the service running
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }
}
```

---

## 4. Enable Sites
Create symlinks from sites-available to sites-enabled:
```bash
sudo ln -s /etc/nginx/sites-available/example1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/example2 /etc/nginx/sites-enabled/
```
Test and reload NGINX:
```bash
sudo nginx -t
sudo systemctl reload nginx
```
If you see:
```bash
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
you are ready to proceed

---

## 5. DNS configuration

Pi-hole must resolve hostnames (e.g., example.home.lan) to the **NGINX server's IP**

In the Pi-hole Admin UI
  1. Go to **Local DNS** -> **DNS Records**
  2. Add entries:
  ```
  Hostname: example1.home.lan
IP: 192.168.1.x

Hostname: example2.home.lan
IP: 192.168.1.x
```
   Make sure the IP address is for the DNS server, and it will be forwarded correctly by NGINX proxy
   
---

## Troubleshooting

| Issue                                 | Cause                     | Fix                                                                                 |
| ------------------------------------- | ------------------------- | ----------------------------------------------------------------------------------- |
| Browser shows “Site can’t be reached” | DNS not resolving         | Check Pi-hole DNS records                                                           |
| `curl` works but browser doesn’t      | Cache or mixed content    | Clear browser cache, disable HTTPS-only mode                                        |
| NGINX gives 502                       | Target server unreachable | Check IP and port of backend service                                                |
| Changes not applied                   | Config not reloaded       | `sudo nginx -t && sudo systemctl reload nginx`                                      |
| Still reaching old IP                 | DNS cache                 | Run `ipconfig /flushdns` (Windows) or `sudo systemd-resolve --flush-caches` (Linux) |

---

## Useful Commands

| Command                                  | Description                     |
| ---------------------------------------- | ------------------------------- |
| `sudo nginx -t`                          | Test NGINX configuration syntax |
| `sudo systemctl reload nginx`            | Reload without downtime         |
| `sudo systemctl restart nginx`           | Full restart                    |
| `sudo journalctl -u nginx`               | View logs                       |
| `sudo tail -f /var/log/nginx/access.log` | Watch requests in real time     |
