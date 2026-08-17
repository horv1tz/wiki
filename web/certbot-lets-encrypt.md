---
description: Бесплатные TLS-сертификаты Let's Encrypt с помощью Certbot
---

# Certbot и Let's Encrypt

**Let's Encrypt** — бесплатный центр сертификации, **Certbot** — официальный клиент для получения и автообновления сертификатов. Сертификаты выдаются на 90 дней, поэтому автоматическое обновление обязательно.

## 1. Установка

```bash
# Debian/Ubuntu — snap-версия рекомендуется официально и всегда свежая
sudo apt install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Плагин для Nginx
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-rfc2136   # пример DNS-плагина (см. раздел про wildcard)
```

Альтернатива из репозитория (обычно старее): `sudo apt install certbot python3-certbot-nginx`.

## 2. Сертификат для сайта на Nginx

Certbot сам отредактирует конфигурацию Nginx и настроит редирект на HTTPS:

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Требование: домен уже указывает на этот сервер, порт 80 открыт, server block с этим `server_name` существует.

Проверка результата — в конфиге появятся строки `ssl_certificate /etc/letsencrypt/live/...` (см. [статью про Nginx](nginx.md)).

## 3. Standalone-режим (без веб-сервера или для других сервисов)

```bash
# Временно поднимает собственный сервер на 80-м порту
sudo certbot certonly --standalone -d mail.example.com
```

## 4. Wildcard-сертификат (*.example.com) через DNS

Wildcard требует подтверждения владения доменом через DNS-запись TXT. Варианты:

### 4.1 Вручную (обновление тоже вручную — неудобно)

```bash
sudo certbot certonly --manual --preferred-challenges dns \
  -d example.com -d "*.example.com"
```

### 4.2 Автоматически через DNS-плагин

Для провайдеров DNS есть плагины (`certbot-dns-cloudflare`, `certbot-dns-rfc2136` и др.). Пример с Cloudflare — файл `/root/.secrets/cloudflare.ini` (права `600`):

```ini
dns_cloudflare_api_token = <api-токен>
```

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d example.com -d "*.example.com"
```

## 5. Автообновление

Certbot устанавливает systemd-таймер автоматически. Проверка:

```bash
systemctl list-timers | grep certbot

# Тест обновления (dry run)
sudo certbot renew --dry-run
```

После обновления часто нужно перезагрузить сервисы. Для Nginx это происходит автоматически (deploy-hook плагина). Для других сервисов добавьте hook:

```bash
# /etc/letsencrypt/renewal-hooks/deploy/reload-services.sh
#!/bin/bash
systemctl reload nginx postfix
```

## 6. Полезные команды

```bash
sudo certbot certificates          # список сертификатов и сроки действия
sudo certbot renew                 # обновить все подходящие по сроку
sudo certbot delete --cert-name example.com   # удалить
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

## 7. Полезные ссылки

* [Certbot — инструкции по ОС и веб-серверу](https://certbot.eff.org/instructions)
* [Документация Let's Encrypt](https://letsencrypt.org/ru/docs/)
