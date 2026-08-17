---
description: >-
  Nexus Sonatype - это инструмент для управления репозиториями, используемый для
  хранения, проксирования и управления артефактами (библиотеки, зависимости,
  Docker-образы и т.д.) в процессе разработки ПО
---

# Nexus Sonatype

## 1. Загрузка Nexus

### 1.1 Скачайте последнюю версию Nexus Repository Manager с официального сайта Sonatype:

```bash
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

### 1.2 Распакуйте архив:

```bash
sudo tar -xvf latest-unix.tar.gz
```

### 1.3 Переименуйте директорию

```bash
cd /opt
sudo mv nexus-3.x.x nexus
```

## 2. Настройка пользователя и прав

### 2.1 Создайте пользователя nexus (для Linux):

```bash
sudo useradd nexus
```

### 2.2 Установите права на директории:

```bash
sudo chown -R nexus:nexus nexus
sudo chown -R nexus:nexus sonatype-work
```

## 3. Конфигурация

### 3.1 Настройте системный сервис

**Создайте файл `/etc/systemd/system/nexus.service`:**

```ini
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

**Перезагрузите конфиги "демонов"**

```
sudo systemctl daemon-reload
```

### 3.2 Создайте файл со следующим содержимым `/opt/nexus.secrets.json`

```
{
  "active": "nexus",
  "keys": [
    {
      "id": "nexus",
      "key": "secret-key"
    }
  ]
}
```

### 3.3 Настройте `nexus-default.properties`

```
# Jetty section
application-port=8081
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature
nexus.secrets.file=/opt/nexus.secrets.json
```

> **Примечание:** строки `nexus-edition` / `nexus-features` и файл секретов относятся к **Nexus Repository Pro** и требуют лицензии. Для бесплатной версии OSS этот блок и файл `/opt/nexus.secrets.json` не нужны — Nexus по умолчанию запустится в редакции OSS.

## 4. Запуск сервиса

### 4.1 Запустите Nexus:

```bash
sudo systemctl start nexus.service
```

### 4.2 Включите автозапуск:

```bash
sudo systemctl enable nexus.service
```

### 4.3 Проверьте статус сервиса:

```bash
sudo systemctl status nexus.service
```

### 4.4 Проверьте логи:

```bash
tail -f /opt/sonatype-work/nexus3/log/nexus.log
```

## 5. Первоначальная настройка

1. Откройте веб-интерфейс по адресу `http://localhost:8081`
2. Войдите с учетными данными по умолчанию:
   * Пользователь: admin
   * Пароль: находится в файле `/opt/sonatype-work/nexus3/admin.password`

## 6. Настройка обратного прокси

### 6.1 Установка и запуск Nginx

**Установка пакета**

```
sudo apt install nginx
```

**Запуск и активация службы**

```bash
sudo systemctl start nginx.service
```

```bash
sudo systemctl enable nginx.service
```

**Проверка статуса сервиса:**

```bash
sudo systemctl status nginx.service
```

### 6.2 Создание файла конфигурации `/etc/nginx/sites-available/nexus.conf`

```nginx
server {
    listen 443 ssl;
    server_name <домен>;

    ssl_certificate /path/to/cert;
    ssl_certificate_key /path/to/key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 10000M;

    location / {
        proxy_pass http://localhost:8081/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port 443;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect http:// https://;
    }
}

server {
    listen 80;
    server_name <домен>;
    return 301 https://$host$request_uri;
}
```

### 6.3 Создание символической ссылки для активации конфигурации

```bash
sudo ln -s /etc/nginx/sites-available/nexus.conf /etc/nginx/sites-enabled/
```

### 6.4 Проверка конфига и перезапуск nginx

**Для проверки:**

```bash
sudo nginx -t
```

**Для перезагрузки конфигурации**

```bash
sudo nginx -s reload
```

или

```bash
sudo systemctl reload nginx.service
```

## 7. Настройка Docker-репозитория

Nexus может выступать приватным Docker Registry (hosted) и кэширующим прокси для Docker Hub (proxy).

### 7.1 Создание репозиториев в веб-интерфейсе

1. **Settings (шестерёнка) > Repository > Repositories > Create repository**
2. **docker (proxy)** — прокси к `https://registry-1.docker.io`, порт коннектора, например 8082
3. **docker (hosted)** — локальный реестр для своих образов, порт коннектора, например 8083
4. **docker (group)** — объединяет proxy и hosted в одну точку входа, порт 8084

> Docker-клиент обращается к репозиториям Nexus через **коннекторы** (отдельные HTTP-порты), а не через основной порт 8081. Откройте эти порты в брандмауэре или настройте виртуальные хосты Nginx для каждого порта.

### 7.2 Realm и аутентификация

Включите **Security > Realms > Docker Bearer Token Realm** (перенести в Active) — иначе `docker login` работать не будет.

### 7.3 Использование

```bash
docker login <домен>:8084
docker tag myapp:latest <домен>:8084/myapp:1.0
docker push <домен>:8084/myapp:1.0

# Через proxy-репозиторий образы Docker Hub тянутся через Nexus
docker pull <домен>:8084/library/nginx
```

## 8. Cleanup Policies — очистка старых артефактов

Чтобы диск не переполнялся, настройте политики очистки:

1. **Settings > Repository > Cleanup Policies > Create Cleanup policy**
2. Формат, например `docker`; критерии: «создан более N дней назад», «не скачивался N дней», «без тега latest»
3. Примените политику в настройках репозитория (**Hosted/Proxy repository > Cleanup Policy**)

Очистка выполняется задачей **Admin > Cleanup unused asset blobs** (запланирована по умолчанию) — она физически удаляет помеченные blob'ы.

## 9. Полезные ссылки

* [Документация Sonatype Nexus](https://help.sonatype.com/en/sonatype-nexus-repository.html)
* [Docker Registry в Nexus](https://help.sonatype.com/en/docker-registry.html)
