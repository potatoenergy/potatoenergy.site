---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "automation", "guide"]
date: "2026-03-17T09:20:00+03:00"
description: "Automatic Docker container shutdown and system power-off on schedule. Scripts, cron setup, recovery."
draft: false
series: ["Docker"]
slug: "docker-scheduled-shutdown-en"
tags: ["docker", "bash", "cron", "automation", "shutdown"]
title: "Docker: Scheduled Automatic Shutdown"
---

For home servers and test environments, resource saving is important: containers can be stopped and system powered off during nights or off-hours. This guide describes a safe method with state preservation and automatic recovery.

> 💡 Method suits OrangePI, Raspberry Pi, old PCs, and any systems where power efficiency matters.

---

## 📦 Container shutdown script

### Create script

```bash
# File: /usr/local/bin/stop_containers_and_shutdown.sh
cat > /usr/local/bin/stop_containers_and_shutdown.sh << 'EOF'
#!/bin/bash
#
# Script saves running container IDs,
# stops them, and initiates system shutdown.
#

CONTAINERS_FILE="/etc/active_containers.txt"

echo "=== $(date '+%Y-%m-%d %H:%M:%S') ==="
echo "Starting container shutdown and system power-off script"

# Get list of running containers (by ID)
RUNNING_CONTAINERS=$(docker ps -q)

if [ -n "${RUNNING_CONTAINERS}" ]; then
    echo "Saving running containers to ${CONTAINERS_FILE}"
    echo "${RUNNING_CONTAINERS}" > "${CONTAINERS_FILE}"
    docker stop ${RUNNING_CONTAINERS}
    echo "Containers stopped."
else
    echo "No running containers."
    [ -f "${CONTAINERS_FILE}" ] && rm -f "${CONTAINERS_FILE}"
fi

sleep 10

echo "Shutting down system."
/sbin/shutdown -h now
EOF
```

### Make executable

```bash
chmod +x /usr/local/bin/stop_containers_and_shutdown.sh
```

### How it works

| Step                           | Description                            |
| ------------------------------ | -------------------------------------- |
| `docker ps -q`                 | Gets IDs of all running containers     |
| `> /etc/active_containers.txt` | Saves list for recovery                |
| `docker stop`                  | Gracefully stops containers (SIGTERM)  |
| `sleep 10`                     | Allows time for operations to complete |
| `shutdown -h now`              | Powers off the system                  |

**Why this way**:

- Saving IDs allows restoring **the same** containers
- `docker stop` gives containers time for graceful shutdown
- `sleep 10` ensures all data is written to disk

---

## 🚀 Container startup script

### Create script

```bash
# File: /usr/local/bin/start_containers.sh
cat > /usr/local/bin/start_containers.sh << 'EOF'
#!/bin/bash
#
# Script reads container list from file,
# starts them, then removes the file.
#

CONTAINERS_FILE="/etc/active_containers.txt"

echo "=== $(date '+%Y-%m-%d %H:%M:%S') ==="
echo "Starting container startup script"

if [ ! -f "${CONTAINERS_FILE}" ]; then
    echo "File ${CONTAINERS_FILE} not found. No startup needed."
    exit 0
fi

CONTAINERS=$(cat "${CONTAINERS_FILE}")
if [ -n "${CONTAINERS}" ]; then
    echo "Found container list: ${CONTAINERS}"
    for cid in ${CONTAINERS}; do
        docker start "${cid}"
        if [ $? -eq 0 ]; then
            echo "Container ${cid} started."
        else
            echo "Error starting container ${cid}."
        fi
    done
else
    echo "File empty. No containers to start."
fi

rm -f "${CONTAINERS_FILE}"
echo "File ${CONTAINERS_FILE} removed."
EOF
```

### Make executable

```bash
chmod +x /usr/local/bin/start_containers.sh
```

### How it works

| Step           | Description                                       |
| -------------- | ------------------------------------------------- |
| File check     | If missing - containers weren't stopped by script |
| `docker start` | Starts containers by saved IDs                    |
| `rm -f`        | Removes file after successful startup             |

**Why remove file**:

- One-time marker - containers won't restart accidentally
- After reboot without script - system runs in normal mode

---

## ⏰ Schedule setup (cron)

### Shutdown (night)

```bash
# crontab -e (as root)
# Shutdown at 00:00 daily
0 0 * * * /usr/local/bin/stop_containers_and_shutdown.sh >> /var/log/docker-shutdown.log 2>&1
```

### Startup (morning)

```bash
# crontab -e (as root)
# Startup at 08:00 daily
0 8 * * * /usr/local/bin/start_containers.sh >> /var/log/docker-startup.log 2>&1
```

### Important notes

| Parameter         | Description                          |
| ----------------- | ------------------------------------ |
| `>> /var/log/...` | Logging for debugging                |
| `2>&1`            | Redirect errors to same log          |
| As root           | Required for `docker` and `shutdown` |

**Verify cron**:

```bash
# Ensure cron is running
systemctl status cron

# View logs
tail -f /var/log/docker-shutdown.log
tail -f /var/log/docker-startup.log
```

---

## 🔧 Alternative: systemd timer (modern method)

### Shutdown timer

```ini
# /etc/systemd/system/docker-shutdown.timer
[Unit]
Description=Daily Docker shutdown timer

[Timer]
OnCalendar=*-*-* 00:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Shutdown service

```ini
# /etc/systemd/system/docker-shutdown.service
[Unit]
Description=Stop Docker containers and shutdown
After=docker.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/stop_containers_and_shutdown.sh
```

### Activate

```bash
systemctl daemon-reload
systemctl enable docker-shutdown.timer
systemctl start docker-shutdown.timer

# Check status
systemctl list-timers | grep docker-shutdown
```

**systemd timer advantages**:

- Precise timing (not affected by cron load)
- Built-in logging via `journalctl`
- `Persistent=true` - runs missed execution after downtime

---

## 🛡 Safety and reliability

### Exclude critical containers

If some containers must run continuously:

```bash
# Modify shutdown script
RUNNING_CONTAINERS=$(docker ps -q --filter "label=shutdown=false")
```

```yaml
# In docker-compose.yml add label
services:
  critical-service:
    image: ...
    labels:
      - "shutdown=false"
```

### Pre-shutdown check

```bash
# Add to shutdown script
ACTIVE_CONNECTIONS=$(netstat -an | grep ESTABLISHED | wc -l)
if [ "$ACTIVE_CONNECTIONS" -gt 10 ]; then
    echo "Active connections detected. Cancelling shutdown."
    exit 1
fi
```

### UPS integration

```bash
# Integration with NUT (Network UPS Tools)
# /etc/nut/upsmon.conf
MONITOR ups@localhost 1 admin password slave
SHUTDOWNCMD "/usr/local/bin/stop_containers_and_shutdown.sh"
```

---

## 📊 Monitoring and debugging

### Check logs

```bash
# Script logs
tail -50 /var/log/docker-shutdown.log
tail -50 /var/log/docker-startup.log

# Systemd logs (if used)
journalctl -u docker-shutdown.service -n 50
journalctl -u docker-startup.service -n 50
```

### Check status

```bash
# Which containers are running
docker ps --format "table {{.Names}}\t{{.Status}}"

# Timer status
systemctl list-timers | grep docker

# State file (if exists)
cat /etc/active_containers.txt 2>/dev/null || echo "File not found"
```

---

## ⚠️ Common issues

```bash
# Script not running via cron
→ Check permissions: ls -la /usr/local/bin/*.sh
→ Check cron: grep CRON /var/log/syslog
→ Ensure cron is running: systemctl status cron

# Containers not starting
→ Check Docker: systemctl status docker
→ Check logs: docker logs <container_id>
→ Verify file exists: cat /etc/active_containers.txt

# System not shutting down
→ Check shutdown available: which shutdown
→ Check permissions: script must run as root
→ View logs: journalctl -b -1 (previous boot)

# File not removed after startup
→ Check file permissions: ls -la /etc/active_containers.txt
→ Remove manually: rm -f /etc/active_containers.txt
```

---

## 🗂 Pre-deployment checklist

- [ ] Tested scripts manually before cron setup
- [ ] Verified logging: `/var/log/docker-*.log`
- [ ] Ensured critical services excluded (if needed)
- [ ] Confirmed container data on persistent volumes
- [ ] Configured startup/shutdown notifications (optional)
- [ ] Created config backup

---

## Links

- 🐳 [Docker Start/Stop Docs](https://docs.docker.com/engine/reference/commandline/start/)
- ⏰ [Cron Documentation](https://man7.org/linux/man-pages/man5/crontab.5.html)
- 🔧 [Systemd Timer Docs](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
- 🛡 [NUT - UPS Management](https://networkupstools.org/)
