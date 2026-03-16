---
author: ["Potato Energy Team", "ponfertato"]
categories: ["openwrt", "networking", "guide"]
date: "2026-03-16T15:25:00+03:00"
description: "Advanced OpenWRT setup: DNS filtering, WireGuard, monitoring, alerts. Configs with explanations."
draft: false
series: ["Networking"]
slug: "setup"
tags: ["openwrt", "wireguard", "dns", "firewall", "prometheus", "telegram"]
title: "OpenWRT: Advanced Router Configuration"
---

This guide covers advanced OpenWRT configuration for power users. We'll set up DNS-level traffic filtering, secure remote access via WireGuard, metrics monitoring, and automated notifications.

> 💡 All sensitive values are replaced with `<...>`. Substitute your own.

---

## 📦 Package installation

```bash
opkg update

# Monitoring: prometheus exporter + modules
opkg install \
  prometheus-node-exporter-lua \
  prometheus-node-exporter-lua-nat_traffic \
  prometheus-node-exporter-lua-netstat \
  prometheus-node-exporter-lua-openwrt \
  prometheus-node-exporter-lua-wifi \
  prometheus-node-exporter-lua-wifi_stations

# WireGuard: kernel module + tools + LuCI
opkg install \
  kmod-wireguard \
  wireguard-tools \
  luci-proto-wireguard

# DNS over HTTPS
opkg install https-dns-proxy

# Debug utilities
opkg install curl jq tmux htop
```

**Why these packages**:

- `prometheus-node-exporter-lua-*` - lightweight Lua modules, minimal resource usage
- `kmod-wireguard` - kernel-space implementation, faster than userspace
- `https-dns-proxy` - encrypts DNS, prevents leaks and censorship

---

## 📢 Event notifications

### On system boot (`/etc/rc.local`)

```bash
sleep 10 && \
curl -s --max-time 10 -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -d chat_id=<CHAT_ID> \
  -d "text=✅ Router: online" >/dev/null 2>&1 && \
curl -s --max-time 10 -H "Content-Type: application/json" -X POST \
  -d '{"content":"✅ Router: online"}' \
  "https://discord.com/api/webhooks/<WEBHOOK_ID>/<WEBHOOK_TOKEN>" >/dev/null 2>&1 && \
exit 0
```

**Why this works**:

- `sleep 10` - lets network initialize before sending
- `--max-time 10` - prevents boot hang if network is down
- Dual channels (Telegram + Discord) - redundancy for critical alerts

### Scheduled reboot (Sunday, 03:00)

```crontab
# /etc/crontabs/root
0 3 * * 0 /usr/bin/curl -s --max-time 10 -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" -d chat_id=<CHAT_ID> -d "text=🔄 Reboot scheduled" >/dev/null 2>&1 && sleep 10 && /sbin/reboot
```

**Why reboot weekly**:

- Clears memory leaks in long-running services
- Night time minimizes user impact
- Pre-reboot alert distinguishes planned vs. crash reboots

---

## 🌐 DNS: smart routing + filtering

### dnsmasq - local resolver

```conf
# /etc/config/dhcp
config dnsmasq
  option cachesize '1000'           # Cache 1000 entries - memory/speed balance
  option local '/lan/'              # .lan domains stay local
  option domain 'lan'               # Default domain for LAN

  # Route specific domains to specific DNS
  list server '/api.github.com/8.8.8.8'   # GitHub via public DNS
  list server '/youtube.com/8.8.8.8'      # YouTube via public DNS

  # Block trackers and "smart" features
  list server '/mask.icloud.com/0.0.0.0'          # Block Private Relay
  list server '/use-application-dns.net/0.0.0.0'  # Prevent DNS bypass

  # Main traffic → local DoH proxy
  list server '127.0.0.1#5053'
  list server '127.0.0.1#5054'
```

**What makes this special**:

- Per-domain upstreams - flexibility and failover
- `0.0.0.0` blocking - trackers never resolve
- Local DoH proxy - encryption without ISP dependency

### https-dns-proxy - DNS over HTTPS

```conf
# /etc/config/https-dns-proxy
config https-dns-proxy
  option resolver_url 'https://<DOH_PROVIDER>/dns-query'
  option listen_port '5053'
  option bootstrap_dns '<RESOLVER_IP>'   # IP to resolve the DoH endpoint itself

config https-dns-proxy
  option resolver_url 'https://<BACKUP_DOH>/dns-query'
  option listen_port '5054'
  option bootstrap_dns '<BACKUP_IP>'
```

**Why two instances**:

- Redundancy: if one provider fails, the other works
- Different `bootstrap_dns` - independence from a single resolver
- Ports 5053/5054 - easy to distinguish in logs and metrics

---

## 🔥 Firewall: "deny all, allow what's needed"

### Base policy

```conf
config defaults
  option input 'REJECT'           # Block inbound by default
  option forward 'REJECT'         # Block forwarding by default
  option output 'ACCEPT'          # Outbound allowed
  option synflood_protect '1'     # SYN-flood protection
  option drop_invalid '1'         # Drop malformed packets
```

**Why this policy**:

- Minimal attack surface: only explicitly allowed traffic passes
- `drop_invalid` - protects against scanning and anomalous traffic

### Zones and rules

```conf
# LAN - trusted
config zone
  option name 'lan'
  list network 'lan'
  option input 'ACCEPT'
  option forward 'ACCEPT'
  option output 'ACCEPT'

# WAN - untrusted
config zone
  option name 'wan'
  list network 'wan' 'wan6'
  option input 'REJECT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'                 # NAT for outbound

# WireGuard - "second LAN"
config zone
  option name 'wg'
  list network 'wg0'
  option input 'ACCEPT'
  option forward 'ACCEPT'
  option output 'ACCEPT'
  option masq '1'
```

**Why the `wg` zone is special**:

- WireGuard clients get LAN access like local devices
- WG→WAN traffic uses the same NAT as LAN→WAN

### Port forwards (templates)

```conf
# WireGuard: inbound UDP 51820 → server in LAN
config redirect
  option name 'wg'
  option src 'wan'
  option proto 'udp'
  option src_dport '51820'
  option dest_ip '<WG_SERVER_LAN_IP>'
  option dest_port '51820'
  option target 'DNAT'

# Game server: arbitrary port → internal host
config redirect
  option name 'game-server'
  option src 'wan'
  option proto 'tcp'
  option src_dport '<EXTERNAL_PORT>'
  option dest_ip '<INTERNAL_IP>'
  option dest_port '<INTERNAL_PORT>'
  option target 'DNAT'
```

**Notes**:

- `option proto` supports lists: `list proto 'tcp'` + `list proto 'udp'`
- Port ranges: `option src_dport '8443-8448'`

### Block QUIC (optional)

```conf
config rule
  option name 'Block-UDP-80'
  option proto 'udp'
  option dest_port '80'
  option target 'REJECT'

config rule
  option name 'Block-UDP-443'
  option proto 'udp'
  option dest_port '443'
  option target 'REJECT'
```

**Why block QUIC**:

- HTTP/3 over QUIC (UDP 443) often bypasses DNS filtering
- Blocking forces clients to fall back to TCP, where filtering works
- Most sites support fallback - no visible impact

---

## 📡 Network: PPPoE, WireGuard, static leases

### Interfaces

```conf
# PPPoE - primary internet (higher priority than default wan)
config interface 'kmtn'
  option proto 'pppoe'
  option device 'eth0.2'
  option username '<PPPOE_LOGIN>'
  option password '<PPPOE_PASSWORD>'
  option metric '0'               # Lower = higher priority

# WireGuard - tunnel interface
config interface 'wg0'
  option proto 'wireguard'
  option private_key '<WG_PRIVATE_KEY>'
  option listen_port '51820'
  list addresses '<WG_SUBNET>/24'

config wireguard_wg0
  option public_key '<PEER_PUBLIC_KEY>'
  option preshared_key '<PRESHARED_KEY>'   # Extra encryption layer
  option endpoint_host '<YOUR_DOMAIN_OR_IP>'
  option endpoint_port '51820'
  option persistent_keepalive '25'         # Keep alive behind NAT
  option route_allowed_ips '1'             # Auto-routing
  list allowed_ips '<PEER_IP>/32'
```

**WireGuard config highlights**:

- `preshared_key` - extra symmetric key, quantum-resistant hedge
- `persistent_keepalive` - essential if router is behind provider NAT
- `route_allowed_ips '1'` - no manual route management needed

### Static DHCP leases

```conf
config host
  option name 'potatoServer'
  option mac '<MAC_ADDRESS>'
  option ip '192.168.1.<STATIC_IP>'
  option leasetime 'infinite'
```

**Why static leases**:

- Servers and critical devices always have the same IP
- Simplifies firewall rules and monitoring
- `leasetime 'infinite'` - lease never expires, even if device is offline

---

## 📊 Monitoring: prometheus-node-exporter

### Configuration

```conf
# /etc/config/prometheus-node-exporter-lua
config prometheus-node-exporter-lua 'main'
  option listen_interface 'lan'   # Listen on LAN only - security
  option listen_port '9101'
```

### Access metrics

```
http://192.168.1.1:9101/metrics
```

### What each module collects

| Module          | Metrics                       | Why it matters               |
| --------------- | ----------------------------- | ---------------------------- |
| `nat_traffic`   | NAT sessions, bytes in/out    | Understand NAT load          |
| `netstat`       | Active connections, states    | Detect anomalies and leaks   |
| `openwrt`       | Version, uptime, packages     | Inventory and alerting       |
| `wifi`          | Signal, channel, airtime load | Optimize wireless            |
| `wifi_stations` | Per-client RSSI, speed, info  | Diagnose problematic devices |

### Grafana PromQL examples

```promql
# CPU usage %
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# WAN traffic (bits/sec)
irate(node_network_receive_bytes_total{device="eth0.2"}[5m]) * 8

# WiFi client count
count(node_wifi_station_info)

# Active NAT sessions
node_nat_traffic_sessions
```

**Why prometheus-node-exporter-lua**:

- Written in Lua - minimal resource footprint
- Integrates with OpenWRT's metric ecosystem
- No separate binary or Python dependency

---

## 📶 WiFi: two networks for different needs

```conf
# 5 GHz - speed
config wifi-iface 'default_radio0'
  option device 'radio0'
  option network 'lan'
  option mode 'ap'
  option ssid '<SSID_5GHZ>'
  option encryption 'psk2'
  option key '<WIFI_PASSWORD>'
  option band '5g'
  option htmode 'VHT80'

# 2.4 GHz - range and compatibility
config wifi-iface 'default_radio1'
  option device 'radio1'
  option network 'lan'
  option mode 'ap'
  option ssid '<SSID_2GHZ>'
  option encryption 'psk2'
  option key '<WIFI_PASSWORD>'
  option band '2g'
  option htmode 'HT40'
```

**Why separate SSIDs**:

- IoT devices often don't support 5 GHz - connect to 2.4
- Manual control: speed where needed (5 GHz), range where needed (2.4)
- Different `htmode`: VHT80 for 5 GHz (wide channel), HT40 for 2.4 (stability)

---

## 🔧 Admin commands cheat sheet

```bash
# WireGuard status (peers, traffic, last handshake)
wg show

# Restart services
/etc/init.d/network restart
/etc/init.d/firewall restart
/etc/init.d/https-dns-proxy restart

# Logs: errors and warnings
logread | grep -iE 'error|warn|fail'

# Recent kernel messages
dmesg | tail -50

# Check listening ports
netstat -tulpn

# Test DNS via specific port
nslookup google.com 127.0.0.1#5053

# Live firewall rule inspection
iptables -L -n -v | grep -E 'REJECT|ACCEPT'
```

---

## ⚠️ Troubleshooting

```bash
# No web UI access
→ Check firewall rule for port 80/443 in 'lan' zone
→ Try SSH: ssh root@192.168.1.1

# WireGuard won't start
→ Verify keys: no spaces, no line breaks, single line
→ Ensure port 51820 is free: netstat -ulpn | grep 51820
→ Check route: `ip route | grep wg0`

# DNS not resolving
→ Proxy status: `/etc/init.d/https-dns-proxy status`
→ Bootstrap must be IP, not domain: `option bootstrap_dns '8.8.8.8'`
→ Test: `nslookup google.com 127.0.0.1#5053`

# Metrics not collected
→ Is port 9101 listening: `netstat -tulpn | grep 9101`
→ Does interface 'lan' exist: `ifconfig | grep br-lan`
→ Check firewall: is inbound on 9101 allowed from 'lan'
```

---

## Links

- 🌐 [OpenWRT Documentation](https://openwrt.org/docs/guide-user/start)
- 🔐 [WireGuard Quick Start](https://www.wireguard.com/quickstart/)
- 🛡 [DNS-over-HTTPS Providers](https://github.com/curl/curl/wiki/DNS-over-HTTPS)
- 📊 [prometheus-node-exporter-lua](https://github.com/openwrt/packages/tree/master/utils/prometheus-node-exporter-lua)
- 🧰 [OpenWRT Package Search](https://openwrt.org/packages/index)
