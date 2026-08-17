---
description: Установка и настройка Prometheus и node_exporter для сбора метрик
---

# Prometheus

**Prometheus** — система мониторинга и база данных временных рядов. Она сама опрашивает (pull) экспортеры метрик по HTTP, хранит данные и позволяет выполнять запросы на языке PromQL. Обычно используется в связке с Grafana для визуализации.

## 1. Установка Prometheus

### 1.1 Создание пользователя и каталогов

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
```

### 1.2 Загрузка и установка

Актуальную версию смотрите на [странице релизов](https://github.com/prometheus/prometheus/releases):

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
tar xvf prometheus-2.54.1.linux-amd64.tar.gz
cd prometheus-2.54.1.linux-amd64

sudo cp prometheus promtool /usr/local/bin/
sudo cp -r consoles console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

### 1.3 Systemd-сервис `/etc/systemd/system/prometheus.service`

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --storage.tsdb.retention.time=30d

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

Веб-интерфейс: `http://<сервер>:9090`.

## 2. Конфигурация `/etc/prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s      # как часто опрашивать цели
  evaluation_interval: 15s  # как часто вычислять правила

rule_files:
  - "rules/*.yml"           # файлы с правилами алертинга

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets:
          - "localhost:9100"
          - "app-server:9100"
          - "db-server:9100"
```

Проверка конфигурации и перечитывание без перезапуска:

```bash
promtool check config /etc/prometheus/prometheus.yml
sudo systemctl reload prometheus   # либо curl -X POST http://localhost:9090/-/reload
```

## 3. node_exporter — метрики сервера

**node_exporter** собирает метрики ОС: CPU, память, диски, сеть и т.д.

### 3.1 Установка

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### 3.2 Systemd-сервис `/etc/systemd/system/node_exporter.service`

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Метрики доступны на `http://<сервер>:9100/metrics`. Не забудьте добавить сервер в `scrape_configs` Prometheus и открыть порт 9100 в брандмауэре только для сервера Prometheus.

## 4. Основы PromQL

Запросы выполняются в веб-интерфейсе Prometheus (вкладка **Graph**) или в Grafana:

```promql
# Загрузка CPU в процентах (по всем ядрам, среднее за 5 минут)
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Свободная память в процентах
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Свободное место на корневом разделе
node_filesystem_avail_bytes{mountpoint="/", fstype!="tmpfs"}

# Сетевой трафик (байт/с)
rate(node_network_receive_bytes_total{device="eth0"}[5m])
```

## 5. Подключение к Grafana

1. **Connections > Data sources > Add data source > Prometheus**
2. URL: `http://<сервер-prometheus>:9090`
3. **Save & test**

Готовые дашборды можно импортировать с [grafana.com/dashboards](https://grafana.com/grafana/dashboards/) — для node_exporter популярен дашборд **1860 (Node Exporter Full)**.

## 6. Полезные ссылки

* [Официальная документация Prometheus](https://prometheus.io/docs/)
* [Каталог экспортеров](https://prometheus.io/docs/instrumenting/exporters/)
* [Alertmanager](alertmanager.md) — маршрутизация уведомлений
