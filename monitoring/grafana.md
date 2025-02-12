---
description: Инструкция по развёртыванию Grafana
---

# Grafana

## Прямая установка на Linux

### Импорт GPG-ключа

Для добавления GPG-ключа Grafana выполните следующую команду:

```bash
wget -qO - https://mirror.half-it.ru/stuff/grafana-gpg-full.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/grafana.gpg > /dev/null
```

### 2. Добавление репозитория Grafana в систему

Откройте файл для добавления источника репозитория Grafana:

```bash
sudo nano /etc/apt/sources.list.d/grafana.list
```

Добавьте в файл следующую строку:

```bash
deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://mirror.half-it.ru/apt-public/apt.grafana.com stable main
```

### Примечания

После добавления репозитория рекомендуется обновить список пакетов с помощью:

```bash
sudo apt update
```

Теперь ваша система настроена для установки и обновления Grafana из указанного репозитория.

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

### Шаг 4: Доступ к Grafana через веб-интерфейс

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
