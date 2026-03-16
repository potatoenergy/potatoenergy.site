---
author: ["Potato Energy Team", "ponfertato"]
categories: ["openwrt", "networking", "guide"]
date: "2026-03-17T00:00:00+03:00"
description: "Профессиональная настройка OpenWRT: DNS-фильтрация, WireGuard, мониторинг, уведомления. Конфиги с пояснениями."
draft: false
series: ["Networking"]
slug: "setup"
tags: ["openwrt", "wireguard", "dns", "firewall", "prometheus", "telegram"]
title: "OpenWRT: Продвинутая настройка роутера"
---

Эта статья - практическое руководство по настройке OpenWRT для продвинутого домашнего использования. Мы разберём конфигурации, которые делают роутер умнее: фильтрация трафика на уровне DNS, безопасный удалённый доступ через WireGuard, мониторинг метрик и автоматические уведомления.

> 💡 Все чувствительные данные заменены на `<...>`. Подставляйте свои значения.

---

## 📦 Установка пакетов

```bash
opkg update

# Мониторинг: prometheus exporter + модули
opkg install \
  prometheus-node-exporter-lua \
  prometheus-node-exporter-lua-nat_traffic \
  prometheus-node-exporter-lua-netstat \
  prometheus-node-exporter-lua-openwrt \
  prometheus-node-exporter-lua-wifi \
  prometheus-node-exporter-lua-wifi_stations

# WireGuard: ядро + утилиты + LuCI
opkg install \
  kmod-wireguard \
  wireguard-tools \
  luci-proto-wireguard

# DNS over HTTPS
opkg install https-dns-proxy

# Утилиты для отладки
opkg install curl jq tmux htop
```

**Зачем**:

- `prometheus-node-exporter-lua-*` - лёгкие модули для сбора метрик без нагрузки на роутер
- `kmod-wireguard` - ядро в kernel-space, работает быстрее пользовательских реализаций
- `https-dns-proxy` - шифрует DNS-запросы, предотвращает утечки и цензуру

---

## 📢 Уведомления о событиях

### При загрузке системы (`/etc/rc.local`)

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

**Зачем**:

- `sleep 10` - даёт время поднять сеть перед отправкой
- `--max-time 10` - не блокирует загрузку, если сеть недоступна
- Два канала (Telegram + Discord) - резервирование уведомлений

### Плановый перезапуск (воскресенье, 03:00)

```crontab
# /etc/crontabs/root
0 3 * * 0 /usr/bin/curl -s --max-time 10 -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" -d chat_id=<CHAT_ID> -d "text=🔄 Reboot scheduled" >/dev/null 2>&1 && sleep 10 && /sbin/reboot
```

**Зачем**:

- Еженедельный ребут очищает утечки памяти в долгосрочной работе
- Ночное время минимизирует влияние на пользователей
- Уведомление «до» помогает отличить плановый ребут от сбоя

---

## 🌐 DNS: умная маршрутизация и фильтрация

### dnsmasq - локальный резолвер

```conf
# /etc/config/dhcp
config dnsmasq
  option cachesize '1000'           # Кэш на 1000 записей - баланс памяти/скорости
  option local '/lan/'              # Локальные домены .lan не уходят в интернет
  option domain 'lan'               # Домен по умолчанию для локальной сети

  # Маршрутизация: конкретные домены → конкретные DNS
  list server '/api.github.com/8.8.8.8'   # GitHub через публичный DNS
  list server '/youtube.com/8.8.8.8'      # YouTube через публичный DNS

  # Блокировка: трекеры и «умные» функции
  list server '/mask.icloud.com/0.0.0.0'          # Блокировка Private Relay
  list server '/use-application-dns.net/0.0.0.0'  # Запрет обхода локального DNS

  # Основной трафик → DoH-прокси (локально)
  list server '127.0.0.1#5053'
  list server '127.0.0.1#5054'
```

**Особенности**:

- Разные upstream'ы для разных доменов - гибкость и отказоустойчивость
- Блокировка на уровне `0.0.0.0` - трекеры не резолвятся вообще
- Локальный DoH-прокси - шифрование без зависимости от провайдера

### https-dns-proxy - DNS over HTTPS

```conf
# /etc/config/https-dns-proxy
config https-dns-proxy
  option resolver_url 'https://<DOH_PROVIDER>/dns-query'
  option listen_port '5053'
  option bootstrap_dns '<RESOLVER_IP>'   # IP для резолвинга самого DoH-эндпоинта

config https-dns-proxy
  option resolver_url 'https://<BACKUP_DOH>/dns-query'
  option listen_port '5054'
  option bootstrap_dns '<BACKUP_IP>'
```

**Зачем два экземпляра**:

- Резервирование: если один провайдер недоступен, работает второй
- Разные `bootstrap_dns` - независимость от одного резолвера
- Порты 5053/5054 - легко различать в логах и метриках

---

## 🔥 Firewall: политика «запретить всё, разрешить нужное»

### Базовые настройки

```conf
config defaults
  option input 'REJECT'           # Блокируем входящие по умолчанию
  option forward 'REJECT'         # Блокируем форвардинг по умолчанию
  option output 'ACCEPT'          # Исходящие - разрешены
  option synflood_protect '1'     # Защита от SYN-flood
  option drop_invalid '1'         # Отбрасываем некорректные пакеты
```

**Почему так**:

- Минимальная поверхность атаки: открыто только то, что явно разрешено
- `drop_invalid` - защита от сканирования и аномального трафика

### Зоны и правила

```conf
# LAN - доверяем
config zone
  option name 'lan'
  list network 'lan'
  option input 'ACCEPT'
  option forward 'ACCEPT'
  option output 'ACCEPT'

# WAN - не доверяем
config zone
  option name 'wan'
  list network 'wan' 'wan6'
  option input 'REJECT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'                 # NAT для исходящих

# WireGuard - как «вторая локалка»
config zone
  option name 'wg'
  list network 'wg0'
  option input 'ACCEPT'
  option forward 'ACCEPT'
  option output 'ACCEPT'
  option masq '1'
```

**Особенность зоны `wg`**:

- WireGuard-клиенты получают доступ к LAN как «свои»
- При этом трафик из WG в WAN идёт через тот же NAT, что и из LAN

### Проброс портов (шаблоны)

```conf
# WireGuard: входящий UDP 51820 → сервер в сети
config redirect
  option name 'wg'
  option src 'wan'
  option proto 'udp'
  option src_dport '51820'
  option dest_ip '<WG_SERVER_LAN_IP>'
  option dest_port '51820'
  option target 'DNAT'

# Игровой сервер: произвольный порт → внутренний хост
config redirect
  option name 'game-server'
  option src 'wan'
  option proto 'tcp'
  option src_dport '<EXTERNAL_PORT>'
  option dest_ip '<INTERNAL_IP>'
  option dest_port '<INTERNAL_PORT>'
  option target 'DNAT'
```

**Важно**:

- `option proto` поддерживает список: `list proto 'tcp'` + `list proto 'udp'`
- Для диапазонов портов: `option src_dport '8443-8448'`

### Блокировка QUIC (опционально)

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

**Зачем блокировать QUIC**:

- HTTP/3 over QUIC (UDP 443) часто обходит DNS-фильтрацию
- Блокировка заставляет клиентов падать на TCP, где фильтрация работает
- Не влияет на большинство сайтов - они поддерживают fallback

---

## 📡 Сеть: PPPoE, WireGuard, статика

### Интерфейсы

```conf
# PPPoE - основной канал (приоритет выше обычного wan)
config interface 'kmtn'
  option proto 'pppoe'
  option device 'eth0.2'
  option username '<PPPOE_LOGIN>'
  option password '<PPPOE_PASSWORD>'
  option metric '0'               # Меньше = выше приоритет

# WireGuard - туннель
config interface 'wg0'
  option proto 'wireguard'
  option private_key '<WG_PRIVATE_KEY>'
  option listen_port '51820'
  list addresses '<WG_SUBNET>/24'

config wireguard_wg0
  option public_key '<PEER_PUBLIC_KEY>'
  option preshared_key '<PRESHARED_KEY>'   # Дополнительный слой шифрования
  option endpoint_host '<YOUR_DOMAIN_OR_IP>'
  option endpoint_port '51820'
  option persistent_keepalive '25'         # Поддержание соединения за NAT
  option route_allowed_ips '1'             # Автоматическая маршрутизация
  list allowed_ips '<PEER_IP>/32'
```

**Особенности WireGuard-конфига**:

- `preshared_key` - защита от будущих квантовых атак (дополнительный симметричный ключ)
- `persistent_keepalive` - критично, если роутер за натом провайдера
- `route_allowed_ips '1'` - не нужно вручную прописывать маршруты

### Статические DHCP-аренды

```conf
config host
  option name 'potatoServer'
  option mac '<MAC_ADDRESS>'
  option ip '192.168.1.<STATIC_IP>'
  option leasetime 'infinite'
```

**Зачем**:

- Сервера и важные устройства всегда по одному адресу
- Упрощает правила фаервола и мониторинг
- `leasetime 'infinite'` - аренда не истекает, даже если устройство офлайн

---

## 📊 Мониторинг: prometheus-node-exporter

### Конфигурация

```conf
# /etc/config/prometheus-node-exporter-lua
config prometheus-node-exporter-lua 'main'
  option listen_interface 'lan'   # Слушаем только локальную сеть - безопасность
  option listen_port '9101'
```

### Доступ к метрикам

```
http://192.168.1.1:9101/metrics
```

### Что собирают модули

| Модуль          | Что измеряет                       | Зачем нужно                      |
| --------------- | ---------------------------------- | -------------------------------- |
| `nat_traffic`   | NAT-сессии, байты в/из             | Понимать нагрузку на трансляцию  |
| `netstat`       | Активные соединения, состояния     | Выявлять аномалии и утечки       |
| `openwrt`       | Версия, аптайм, пакеты             | Инвентаризация и алертинг        |
| `wifi`          | Сигнал, канал, загрузка эфира      | Оптимизация беспроводной сети    |
| `wifi_stations` | Клиенты, RSSI, скорость на каждого | Диагностика проблемных устройств |

### Примеры PromQL-запросов для Grafana

```promql
# Загрузка CPU в %
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Трафик на WAN (бит/с)
irate(node_network_receive_bytes_total{device="eth0.2"}[5m]) * 8

# Количество клиентов WiFi
count(node_wifi_station_info)

# NAT-сессии в реальном времени
node_nat_traffic_sessions
```

**Почему prometheus-node-exporter-lua**:

- Написан на Lua - минимальное потребление ресурсов
- Интегрируется в OpenWRT-систему метрик
- Не требует отдельного бинарника или Python

---

## 📶 WiFi: две сети для разных задач

```conf
# 5 ГГц - скорость
config wifi-iface 'default_radio0'
  option device 'radio0'
  option network 'lan'
  option mode 'ap'
  option ssid '<SSID_5GHZ>'
  option encryption 'psk2'
  option key '<WIFI_PASSWORD>'
  option band '5g'
  option htmode 'VHT80'

# 2.4 ГГц - дальность и совместимость
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

**Зачем разные SSID**:

- IoT-устройства часто не поддерживают 5 ГГц - подключаются к 2.4
- Ручной выбор: где нужна скорость - 5 ГГц, где важна дальность - 2.4
- Разные `htmode`: VHT80 для 5 ГГц (широкий канал), HT40 для 2.4 (стабильность)

---

## 🔧 Полезные команды для администрирования

```bash
# Статус WireGuard (пиры, трафик, last handshake)
wg show

# Перезапуск служб
/etc/init.d/network restart
/etc/init.d/firewall restart
/etc/init.d/https-dns-proxy restart

# Логи: ошибки и предупреждения
logread | grep -iE 'error|warn|fail'

# Последние сообщения ядра
dmesg | tail -50

# Проверка занятых портов
netstat -tulpn

# Тест DNS через конкретный порт
nslookup google.com 127.0.0.1#5053

# Проверка правил фаервола в реальном времени
iptables -L -n -v | grep -E 'REJECT|ACCEPT'
```

---

## ⚠️ Диагностика: если что-то не работает

```bash
# Нет доступа к веб-интерфейсу
→ Проверь правило для порта 80/443 в зоне 'lan'
→ Попробуй по SSH: ssh root@192.168.1.1

# WireGuard не поднимается
→ Проверь ключи: без пробелов, переносов, в одной строке
→ Убедись, что порт 51820 свободен: netstat -ulpn | grep 51820
→ Проверь маршрут: `ip route | grep wg0`

# DNS не резолвит
→ Статус прокси: `/etc/init.d/https-dns-proxy status`
→ Bootstrap должен быть IP, не домен: `option bootstrap_dns '8.8.8.8'`
→ Тест: `nslookup google.com 127.0.0.1#5053`

# Метрики не собираются
→ Слушает ли порт 9101: `netstat -tulpn | grep 9101`
→ Существует ли интерфейс 'lan': `ifconfig | grep br-lan`
→ Проверь фаервол: разрешён ли входящий на 9101 из 'lan'
```

---

## Ссылки

- 🌐 [OpenWRT Documentation](https://openwrt.org/docs/guide-user/start)
- 🔐 [WireGuard Quick Start](https://www.wireguard.com/quickstart/)
- 🛡 [DNS-over-HTTPS Providers](https://github.com/curl/curl/wiki/DNS-over-HTTPS)
- 📊 [prometheus-node-exporter-lua](https://github.com/openwrt/packages/tree/master/utils/prometheus-node-exporter-lua)
- 🧰 [OpenWRT Package Search](https://openwrt.org/packages/index)
