# Linux Ops Homelab — Operational Runbook

**Server:** saikalyan.website  
**OS:** Ubuntu 24.04 LTS  
**Managed by:** saiops  
**Last updated:** June 2026

---

## 1. SSH Access

```bash
ssh -i ~/Desktop/homelab-key.pem saiops@YOUR_EC2_PUBLIC_IP
```

- Root login is disabled
- Password login is disabled — key only
- fail2ban bans IPs after 3 failed attempts for 1 hour

---

## 2. Nginx — Reload After Config Change

Always test before reloading:

```bash
sudo nginx -t                    # test config syntax
sudo systemctl reload nginx      # apply changes, zero downtime
sudo systemctl status nginx      # verify running
```

Never use `restart` unless `reload` fails — restart causes brief downtime.

---

## 3. TLS Certificate Rotation

Certbot auto-renews via systemd timer. To check renewal status:

```bash
sudo systemctl status certbot.timer
sudo certbot renew --dry-run     # test renewal without actually renewing
```

To manually force renewal:

```bash
sudo certbot renew --force-renewal
sudo systemctl reload nginx
```

Cert location: `/etc/letsencrypt/live/saikalyan.website/`  
Cert expires: 2026-09-04 (auto-renews 30 days before expiry)

---

## 4. Disk Full Response

Check disk usage:

```bash
df -h                            # overview of all partitions
du -sh /var/log/nginx/*          # check nginx log sizes
du -sh /home/saiops/*            # check home directory
```

Free up space:

```bash
sudo journalctl --vacuum-time=7d     # delete system logs older than 7 days
sudo apt autoremove -y               # remove unused packages
sudo rm -f /home/saiops/node_exporter-1.8.1.linux-amd64.tar.gz  # remove installer
```

Alert threshold: investigate at 80%, act at 90%.

---

## 5. fail2ban — Unban an IP

Check banned IPs:

```bash
sudo fail2ban-client status sshd
```

Unban a specific IP:

```bash
sudo fail2ban-client set sshd unbanip YOUR_IP
```

If you locked yourself out — use AWS Console EC2 Instance Connect as emergency access.

---

## 6. Service Health Check

Quick status of all services:

```bash
sudo systemctl status nginx
sudo systemctl status fail2ban
sudo systemctl status node_exporter
sudo systemctl status unattended-upgrades
sudo ufw status verbose
```

---

## 7. Host Metrics Baseline (node_exporter)

Access metrics endpoint:

```bash
curl -s http://localhost:9100/metrics | grep "node_cpu_seconds_total" | head -5
curl -s http://localhost:9100/metrics | grep "node_filesystem_avail_bytes"
curl -s http://localhost:9100/metrics | grep "node_memory_MemAvailable_bytes"
```

Normal baseline (recorded June 2026):

- CPU idle: ~28,000s (very low load)
- Disk available on /: ~4.4GB (35% used)
- Memory available: ~544MB

---

## 8. Disk Usage Log

Cron logs disk usage every 6 hours to:

```bash
cat /home/saiops/disk_usage.log
```

Cron schedule: `0 */6 * * *` (midnight, 6am, 12pm, 6pm UTC)
