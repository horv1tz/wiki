---
description: Сбор логов с помощью Loki и Promtail с просмотром в Grafana
---

# Loki и Promtail

**Loki** — система агрегации логов от Grafana Labs, «Prometheus для логов»: логи индексируются только по меткам (labels), что делает её легче и дешевле Elasticsearch. **Promtail** — агент, который читает логи на серверах и отправляет их в Loki.

## 1. Установка Loki

Репозиторий Grafana настраивается так же, как описано в [статье про Grafana](grafana.md). Затем:

```bash
sudo apt install loki
```

### Конфигурация `/etc/loki/config.yml`

Минимальная конфигурация для одного сервера (локальное хранилище):

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  instance_addr: 127.0.0.1
  path_prefix: /var/lib/loki
  storage:
    filesystem:
      chunks_directory: /var/lib/loki/chunks
      rules_directory: /var/lib/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

limits_config:
  retention_period: 720h   # хранить логи 30 дней

compactor:
  working_directory: /var/lib/loki/compactor
  retention_enabled: true
  delete_request_store: filesystem
```

```bash
sudo systemctl enable --now loki
```

## 2. Установка Promtail (на каждом сервере, с которого собираем логи)

```bash
sudo apt install promtail
```

### Конфигурация `/etc/promtail/config.yml`

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /var/lib/promtail/positions.yaml   # где promtail запоминает, до куда дочитал

clients:
  - url: http://<сервер-loki>:3100/loki/api/v1/push

scrape_configs:
  # Логи systemd (journald)
  - job_name: journal
    journal:
      max_age: 12h
      labels:
        job: systemd-journal
        host: <имя-сервера>
    relabel_configs:
      - source_labels: ["__journal__systemd_unit"]
        target_label: unit

  # Логи из файлов (например, nginx)
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          host: <имя-сервера>
          __path__: /var/log/nginx/*.log
```

```bash
sudo systemctl enable --now promtail
```

## 3. Подключение Loki к Grafana

1. **Connections > Data sources > Add data source > Loki**
2. URL: `http://<сервер-loki>:3100`
3. **Save & test**

## 4. Основы LogQL

Запросы выполняются в Grafana (**Explore**, источник Loki):

```logql
# Все логи конкретного systemd-сервиса
{job="systemd-journal", unit="nginx.service"}

# Только строки с ошибками
{job="nginx"} |= "error"

# Ошибки 5xx из access-лога nginx (регулярное выражение)
{job="nginx"} |~ "HTTP/\\S+\" 5\\d\\d"

# Количество ошибок в минуту (для графиков и алертов)
sum(count_over_time({job="nginx"} |= "error" [1m]))
```

## 5. Полезные ссылки

* [Документация Loki](https://grafana.com/docs/loki/latest/)
* [Документация Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/)
* [Справочник по LogQL](https://grafana.com/docs/loki/latest/logql/)
