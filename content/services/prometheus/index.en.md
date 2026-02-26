---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Monitoring", "Security"]
description: "Comprehensive infrastructure surveillance system"
slug: "prometheus"
title: "Prometheus: Infrastructure's All-Seeing Eye"
---

### Prometheus: Infrastructure monitoring [📊](https://prometheus.io/)

**Warning system** about problems before they occur.

**What controls:**

- 🖥️ Server metrics: CPU, RAM, disk, network via node_exporter
- 🌐 Service availability: blackbox-HTTP/TCP/ICMP checks
- 📈 Collecting metrics from applications: Nextcloud, Home Assistant, and others
- 🚨 Alertmanager: notifications in Telegram/Discord when thresholds are exceeded
- 🔍 Powerful queries via PromQL for deep analysis

**How it works:**

1. Prometheus collects metrics on a schedule (scrap)
2. Do you see graphs in Grafana or set up your dashboards
3. In case of an anomaly, an alert is triggered and the administrator receives a notification.

**For administrators:**
Flexible alert rules, recording rules, federation of metrics, long-term storage via Thanos.

**Access:** via Grafana (`grafana.potatoenergy.ru`) • according to Potato Energy credentials (management is only based on the rights of the `admin` group)
