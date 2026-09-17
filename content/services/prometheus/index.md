---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Мониторинг", "Безопасность"]
description: "Комплексная система наблюдения за инфраструктурой"
slug: "prometheus"
title: "Prometheus: Сбор метрик"
---

### Prometheus: Сбор метрик [📊](https://prometheus.io/)

**Система сбора метрик** для предупреждения о проблемах до их возникновения.

**Функции:**

- 🖥️ Метрики серверов: CPU, RAM, диск, сеть через node_exporter
- 🌐 Доступность сервисов: blackbox-проверки HTTP/TCP/ICMP
- 📈 Сбор метрик из приложений: Nextcloud, Home Assistant и других
- 🚨 Alertmanager: уведомления в Telegram/Discord при превышении порогов
- 🔍 Запросы через PromQL для глубокого анализа

**Принцип работы:**

1. Prometheus собирает метрики по расписанию (scrape)
2. Графики отображаются в Grafana
3. При аномалии срабатывает алерт - администратор получает уведомление

**Для администраторов:**
Правила алертинга, recording rules, федерация метрик, долгосрочное хранение через Thanos.

**Доступ:** через Grafana (`grafana.potatoenergy.ru`) • по учётным данным Potato Energy (управление - только по правам группы `admin`)
