---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Monitoring", "Security"]
description: "Comprehensive infrastructure surveillance system"
slug: "prometheus"
title: "Prometheus: Metrics Collection"
---

### Prometheus: Metrics Collection [📊](https://prometheus.io/)

**Metrics collection system** for early warning about problems.

**Functions:**

- 🖥️ Server metrics: CPU, RAM, disk, network via node_exporter
- 🌐 Service availability: blackbox HTTP/TCP/ICMP checks
- 📈 Collecting metrics from applications: Nextcloud, Home Assistant, and others
- 🚨 Alertmanager: notifications in Telegram/Discord when thresholds are exceeded
- 🔍 Queries via PromQL for deep analysis

**How it works:**

1. Prometheus collects metrics on a schedule (scrape)
2. Charts are displayed in Grafana
3. On anomaly, an alert is triggered - the administrator receives a notification

**For administrators:**
Alert rules, recording rules, federation of metrics, long-term storage via Thanos.

**Access:** via Grafana (`grafana.potatoenergy.ru`) • according to Potato Energy credentials (management is only based on the rights of the `admin` group)
