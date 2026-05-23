# Server Monitoring

Commands to check what's happening on your server — CPU, memory, disk, processes, and logs.

---

## Check System Resources

### CPU and Memory

```bash
# Live interactive view (press q to quit)
htop

# Quick memory check
free -h

# Check CPU count
nproc
```

### Disk Space

```bash
# Check all drives/partitions
df -h

# Find what's using the most space
du -sh /var/www/* | sort -rh | head -10
du -sh /var/log/* | sort -rh | head -10
```

### Check if Server is Overloaded

```bash
# Server load (1-min, 5-min, 15-min averages)
# Numbers should be below your CPU count
uptime
```

---

## Check Running Services

```bash
# List all running services
sudo systemctl list-units --type=service --state=running

# Check a specific service
sudo systemctl status nginx
sudo systemctl status pm2-deployer  # or your PM2 service name
```

---

## Check Open Ports

See what's listening on the network. Unexpected open ports are a security risk.

```bash
sudo ss -tlnp
```

You should only see ports you expect (22/2222, 80, 443, 3000, etc.).

---

## Check Logs

### Nginx Logs

```bash
# Live access log (see who's visiting)
sudo tail -f /var/log/nginx/access.log

# Error log (see what's broken)
sudo tail -f /var/log/nginx/error.log
```

### System Logs

```bash
# Recent system events
sudo journalctl -xe

# Follow system log live
sudo journalctl -f

# See who logged in
last
sudo grep "Failed password" /var/log/auth.log | tail -20
```

### App Logs (PM2)

```bash
pm2 logs myapp
pm2 logs myapp --lines 100
```

---

## Check Network

```bash
# See all active connections
ss -tn

# Check if a port is open from outside (run on your local machine)
nc -zv YOUR_SERVER_IP 80
nc -zv YOUR_SERVER_IP 443
```

---

## Check Disk I/O (If Server is Slow)

```bash
# Install if not available
sudo apt install iotop -y

# See what's reading/writing to disk
sudo iotop
```

---

## Set Up a Simple Alert (Disk Space)

Get an email if disk is over 80% full. Add this to cron:

```bash
crontab -e
```

Add:
```
0 8 * * * df -h | awk '$5+0 > 80 {print "WARNING: Disk usage over 80% on " $6 " (" $5 ")" }' | mail -s "Disk Alert" your@email.com
```

---

## Quick Health Check Script

Save this as `/usr/local/bin/check-server.sh`:

```bash
#!/bin/bash
echo "=== Server Health Check ==="
echo ""
echo "Uptime:"
uptime
echo ""
echo "Memory:"
free -h
echo ""
echo "Disk:"
df -h /
echo ""
echo "PM2 Apps:"
pm2 list
echo ""
echo "Nginx Status:"
sudo systemctl is-active nginx
```

```bash
chmod +x /usr/local/bin/check-server.sh
check-server.sh
```

---

## Quick Reference

| What to Check | Command |
|---------------|---------|
| CPU/Memory live | `htop` |
| Disk space | `df -h` |
| Open ports | `sudo ss -tlnp` |
| Nginx errors | `sudo tail -f /var/log/nginx/error.log` |
| App logs | `pm2 logs myapp` |
| Failed logins | `sudo grep "Failed" /var/log/auth.log` |
| Server load | `uptime` |
| Who's logged in | `who` |
