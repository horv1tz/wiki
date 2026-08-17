---
description: Инструкция по развёртыванию Grafana
---

# Grafana

## Прямая установка на Linux

### Шаг 1: Импорт GPG-ключа

Создайте директорию для ключей и импортируйте GPG-ключ репозитория Grafana:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
wget -qO - https://mirror.half-it.ru/stuff/grafana-gpg-full.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

### Шаг 2: Добавление репозитория Grafana в систему

Создайте файл источника репозитория:

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://mirror.half-it.ru/apt-public/apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

> **Важно:** путь в `signed-by` должен совпадать с путём, куда сохранён ключ (`/etc/apt/keyrings/grafana.gpg`).

### Шаг 3: Обновление списка пакетов

```bash
sudo apt update
```

Теперь система настроена для установки и обновления Grafana из указанного репозитория.

## Установка Grafana на Linux

В данном разделе описывается процесс установки пакета **Grafana** на систему Linux после добавления репозитория Grafana, как описано в предыдущей документации.

### Шаг 1: Установка пакета Grafana

Для установки Grafana выполните следующую команду:

```bash
sudo apt install grafana
```

### Шаг 2: Запуск и настройка автозапуска сервиса Grafana

После установки необходимо запустить сервис Grafana и настроить его автоматический запуск при загрузке системы.

#### Запуск сервиса Grafana:

```bash
sudo systemctl start grafana-server
```

#### Проверка статуса сервиса:

```bash
sudo systemctl status grafana-server
```

Вы должны увидеть статус `active (running)`, что означает успешный запуск Grafana.

#### Включение автозапуска Grafana при загрузке системы:

```bash
sudo systemctl enable grafana-server
```

### Шаг 3: Доступ к Grafana через веб-интерфейс

После успешной установки и запуска сервиса Grafana, вы можете получить доступ к веб-интерфейсу для дальнейшей настройки и использования.

1. Откройте веб-браузер.
2. Перейдите по адресу: `http://localhost:3000`
   * Если вы устанавливаете Grafana на удалённый сервер, замените `localhost` на IP-адрес или доменное имя вашего сервера, например: `http://your-server-ip:3000`
3.  Введите учётные данные для входа:

    * **Имя пользователя:** `admin`
    * **Пароль:** `admin`

    При первом входе система запросит смену пароля.

### Дополнительные команды и настройки

#### Перезапуск сервиса Grafana

Если необходимо перезапустить сервис Grafana после внесения изменений в конфигурацию:

```bash
sudo systemctl restart grafana-server
```

#### Остановка сервиса Grafana

Для остановки сервиса Grafana:

```bash
sudo systemctl stop grafana-server
```

#### Проверка версии Grafana

Для проверки установленной версии Grafana используйте команду:

```bash
grafana-server -v
```

## Provisioning — источники данных и дашборды как код

Вместо ручной настройки через веб-интерфейс источники данных и дашборды можно описывать файлами — они подхватятся при запуске Grafana и удобно версионируются в Git.

### Источник данных `/etc/grafana/provisioning/datasources/prometheus.yml`

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://localhost:9090
    isDefault: true

  - name: Loki
    type: loki
    access: proxy
    url: http://localhost:3100
```

### Дашборды `/etc/grafana/provisioning/dashboards/dashboards.yml`

```yaml
apiVersion: 1

providers:
  - name: default
    folder: ""
    type: file
    options:
      path: /etc/grafana/dashboards
```

JSON-файлы дашбордов кладутся в `/etc/grafana/dashboards/`. Экспортировать существующий дашборд: **Dashboard settings > JSON Model**.

```bash
sudo systemctl restart grafana-server
```

## Полезные ссылки

* [Официальная документация Grafana](https://grafana.com/docs/grafana/latest/)
* [Provisioning Grafana](https://grafana.com/docs/grafana/latest/administration/provisioning/)
* [Grafana Dashboards](https://grafana.com/grafana/dashboards/) — каталог готовых дашбордов
