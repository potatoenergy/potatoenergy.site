---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Monitoring", "Security"]
description: "Comprehensive infrastructure surveillance system"
slug: "prometheus-stack-monitoring"
title: "Prometheus: the all-seeing eye of infrastructure"
---

### Monitoring Platform [📊](https://prometheus.io/)

**Purpose** 
24/7 monitoring of key indicators:
- Service availability (HTTP/ICMP/DNS)
- Resource utilization (CPU/RAM/Disk)
- Abnormal activity
- Execution SLA

**Technical Implementation**  
- Metrics collection: 20s interval
- Storage: 30 days retention
- Samples: Blackbox for 8 types of tests
- Exporters: Node, cAdvisor, ASF, HA

**Security and Access**  
- Dashboard: `potatoenergy.ru/prometheus` (group `dev`)
- Alerts: Discord/Telegram for critical incidents
- Encryption: TLS for all exporters
- Audit: Signature metrics

**Features**  
- Automatic anomaly detection
- Grafana custom dashboards
- Integration with 15+ data sources
- Incident Escalation System

---

### Alerting System [🚨](https://prometheus.io/docs/alerting/latest/alertmanager/)

**Principles of Operation** 
1. 200+ pre-defined rules for:
   - Service availability
   - Resource utilization thresholds
   - Network traffic anomalies
   - Application errors

2. Multilayer routing:
```yaml
route:
  receiver: grafana
  routes:
    - match: severity=critical
      receivers: [discord, telegram]
````

3. notifications for the `dev` group only:
- Discord: Channel #infra-alerts
- Telegram: Private channel with bot.
- Escalation after 30 minutes without confirmation

---

#Why is this important?
1. **Proactive detection** of issues before impacting users
2. **Single point of truth** for analyzing incidents
3. **Automated documentation** via tags
4. **Resource optimization** through historical data

All non-critical alerts are handled during project business hours.
