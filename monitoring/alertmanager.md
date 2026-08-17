---
description: Настройка Alertmanager для маршрутизации уведомлений из Prometheus
---

# Alertmanager

**Alertmanager** принимает алерты от Prometheus, группирует и дедуплицирует их, а затем рассылает уведомления в Telegram, email, Slack и другие каналы.

## 1. Установка

```bash
sudo useradd --no-create-home --shell /bin/false alertmanager
sudo mkdir /etc/alertmanager /var/lib/alertmanager

cd /tmp
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar xvf alertmanager-0.27.0.linux-amd64.tar.gz
sudo cp alertmanager-0.27.0.linux-amd64/alertmanager /usr/local/bin/
sudo cp alertmanager-0.27.0.linux-amd64/amtool /usr/local/bin/
```

Systemd-сервис `/etc/systemd/system/alertmanager.service`:

```ini
[Unit]
Description=Alertmanager
After=network.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now alertmanager
```

Веб-интерфейс: `http://<сервер>:9093`.

## 2. Правила алертинга в Prometheus

Создайте файл `/etc/prometheus/rules/node.yml`:

```yaml
groups:
  - name: node
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Инстанс {{ $labels.instance }} недоступен"

      - alert: HighCPU
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Высокая загрузка CPU на {{ $labels.instance }}"

      - alert: DiskAlmostFull
        expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "На {{ $labels.instance }} заканчивается место ({{ $labels.mountpoint }})"
```

Подключите Alertmanager в `/etc/prometheus/prometheus.yml`:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["localhost:9093"]
```

```bash
promtool check rules /etc/prometheus/rules/node.yml
sudo systemctl reload prometheus
```

## 3. Конфигурация Alertmanager

### 3.1 Уведомления в Telegram

1. Создайте бота через [@BotFather](https://t.me/BotFather) и получите токен
2. Узнайте `chat_id` чата (например, через [@userinfobot](https://t.me/userinfobot))

Файл `/etc/alertmanager/alertmanager.yml`:

```yaml
global:
  telegram_api_url: "https://api.telegram.org"

route:
  receiver: "telegram"
  group_by: ["alertname", "instance"]
  group_wait: 30s        # ожидание перед отправкой группы алертов
  group_interval: 5m     # интервал между повторными уведомлениями группы
  repeat_interval: 4h    # повтор уведомления, если алерт не resolved

receivers:
  - name: "telegram"
    telegram_configs:
      - bot_token: "<токен-бота>"
        chat_id: <chat_id>
        message: |
          {{ range .Alerts }}
          <b>{{ .Annotations.summary }}</b>
          Статус: {{ .Status }}
          {{ end }}
        parse_mode: HTML
```

### 3.2 Уведомления на email

```yaml
receivers:
  - name: "email"
    email_configs:
      - to: "admin@example.com"
        from: "alertmanager@example.com"
        smarthost: "smtp.example.com:587"
        auth_username: "alertmanager@example.com"
        auth_password: "<пароль>"
```

Применение и проверка:

```bash
sudo systemctl restart alertmanager
amtool config show            # показать текущую конфигурацию
```

## 4. Управление алертами (amtool)

```bash
# Список активных алертов
amtool alert query

# «Заглушить» алерт на 2 часа
amtool silence add alertname=HighCPU instance=app-server:9100 --duration=2h --comment="Плановые работы"

# Список заглушений и их удаление
amtool silence query
amtool silence expire <id>
```

## 5. Полезные ссылки

* [Документация Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
* [Настройка правил алертинга](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
