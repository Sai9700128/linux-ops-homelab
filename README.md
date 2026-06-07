# Linux Ops Homelab

A production-grade hardened Ubuntu 24.04 EC2 server serving a static site over HTTPS — built to demonstrate end-to-end Linux server ownership beyond container-level operations.

**Live:** <https://saikalyan.website>

---

## Architecture

```
Internet → AWS Security Group (ports 22/80/443)
                ↓
         Ubuntu 24.04 EC2 (t2.micro)
                ↓
         UFW Host Firewall (deny by default)
                ↓
         Nginx (reverse proxy + TLS termination)
                ↓
         /var/www/saikalyan.website (static site)
```

**Security Layer:**

- fail2ban (SSH brute force protection)
- SSH key-only auth, root login disabled
- Nginx security headers (HSTS, X-Frame-Options, nosniff)
- Let's Encrypt TLS with auto-renewal

**Observability Layer:**

- node_exporter (host metrics on :9100)
- cron disk usage logging every 6hrs
- Nginx access/error logs with 14-day logrotate

---

## Tech Stack

| Layer | Tool |
|-------|------|
| Cloud | AWS EC2 (t2.micro, Ubuntu 24.04) |
| Web server | Nginx |
| TLS | Let's Encrypt via certbot |
| Host firewall | UFW |
| Brute force protection | fail2ban |
| Host metrics | node_exporter |
| Log management | logrotate + journald |
| Scheduled tasks | cron |
| Auto security patches | unattended-upgrades |

---

## Security Hardening Checklist

- [x] Non-root sudo user (`saiops`)
- [x] SSH key-only authentication
- [x] Root login disabled
- [x] UFW firewall — deny all incoming except 22/80/443
- [x] fail2ban SSH jail — ban after 3 failed attempts
- [x] Nginx version disclosure disabled
- [x] Security headers — HSTS, X-Frame-Options, nosniff
- [x] Automatic security patches via unattended-upgrades
- [x] TLS auto-renewal via systemd timer

---

## Observability

node_exporter exposes 600+ host metrics at `:9100/metrics`

Normal baseline (June 2026):

- CPU idle: ~28,000s (low load)
- Disk available: ~4.4GB (35% used)
- Memory available: ~544MB

---

## Operational Runbook

See [RUNBOOK.md](./RUNBOOK.md) for:

- SSH access procedure
- Nginx reload and config testing
- TLS cert rotation
- Disk full response
- fail2ban IP unban procedure
- Service health checks
