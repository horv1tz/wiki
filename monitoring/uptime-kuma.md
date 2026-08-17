---
description: Uptime Kuma — простой self-hosted мониторинг доступности сервисов
---

# Uptime Kuma

**Uptime Kuma** — лёгкий self-hosted инструмент мониторинга доступности: HTTP(s), TCP, ping, DNS, Docker-контейнеры и т.д. Поддерживает статус-страницы и уведомления (Telegram, email, Slack и 90+ сервисов). Хорошо дополняет Prometheus-стек простыми проверками «извне».

## 1. Установка через Docker Compose

Файл `compose.yaml`:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - uptime-kuma-data:/app/data

volumes:
  uptime-kuma-data:
```

```bash
docker compose up -d
```

Веб-интерфейс: `http://<сервер>:3001`. При первом входе создайте учётную запись администратора.

## 2. Основные возможности

* **Мониторы**: HTTP(s) с проверкой ключевого слова и сертификата, TCP-порт, ping, DNS-запись, Steam Game Server, Docker-контейнер
* **Уведомления**: Telegram, email (SMTP), Slack, Discord, Gotify и десятки других сервисов
* **Статус-страницы**: публичные страницы с текущим состоянием сервисов и историей инцидентов
* **Сертификаты**: отслеживание срока действия TLS-сертификатов с предупреждением

## 3. Пример настройки монитора

1. **Add New Monitor**
2. Тип: `HTTP(s)`, URL: `https://example.com`, интервал: `60s`
3. Включите **Certificate Expiry Notification** для отслеживания сертификата
4. В разделе **Settings > Notifications** добавьте Telegram-бота и привяжите уведомление к монитору

## 4. Обратный прокси (Nginx)

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name status.example.com;

    ssl_certificate     /etc/letsencrypt/live/status.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/status.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # WebSocket необходим для работы интерфейса
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

## 5. Обновление

```bash
docker compose pull
docker compose up -d
```

## 6. Полезные ссылки

* [GitHub Uptime Kuma](https://github.com/louislam/uptime-kuma)
