---
description: Подробная инструкция по установке и настройке GitLab Runner для Debian/Ubuntu
---

# Установка и настройка Gitlab Runner

# 1. Установка GitLab Runner

Для начала нам необходимо установить GitLab Runner на сервер. Выполните следующие команды последовательно:

```bash
# Обновляем список пакетов
sudo apt-get update

# Устанавливаем curl для загрузки скрипта установки
sudo apt-get install -y curl

# Добавляем официальный репозиторий GitLab
# Этот скрипт автоматически добавит GPG ключи и настроит репозиторий
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# Устанавливаем GitLab Runner
sudo apt-get install gitlab-runner

# Проверяем успешность установки
gitlab-runner --version
```

После установки GitLab Runner будет автоматически запущен как системный сервис.

# 2. Регистрация Runner

## Актуальный способ: runner authentication token (GitLab 16+)

Начиная с GitLab 16.0 регистрация через legacy registration token объявлена устаревшей (удалена в GitLab 18.0). Вместо него используются **runner authentication tokens** (префикс `glrt-`), которые создаются в веб-интерфейсе:

1. Войдите в веб-интерфейс GitLab
2. Перейдите в **Admin > CI/CD > Runners** (для instance runner) или **Settings > CI/CD > Runners** внутри группы/проекта
3. Нажмите **New instance runner** / **New project runner**
4. Укажите теги, описание и другие параметры, нажмите **Create runner**
5. Скопируйте выданный токен вида `glrt-...`

Затем зарегистрируйте runner на сервере:

```bash
sudo gitlab-runner register \
  --url "https://gitlab.example.com/" \
  --token "glrt-xxxxxxxxxxxx" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "production-server-01"
```

Интерактивный вариант (`sudo gitlab-runner register`) также работает, но в качестве токена запросит именно authentication token.

## Legacy-способ (GitLab 15.x и старше)

Если ваш GitLab старше 16-й версии, регистрационный token находится в **Settings > CI/CD > Runners**, раздел "Set up a specific Runner manually". Запустите:

```bash
sudo gitlab-runner register
```

Мастер попросит ввести:

```bash
# 1. GitLab instance URL
# Например: https://gitlab.com/ или https://gitlab.your-domain.com/

# 2. Registration token

# 3. Описание runner'а
# Например: "production-server-01" или "docker-runner-staging"

# 4. Теги (опционально)
# Например: docker,aws,production

# 5. Выбор executor'а
# - shell (выполняет команды напрямую в системе)
# - docker (запускает задачи в Docker контейнерах)
# - docker-windows (для Windows контейнеров)
# - docker+machine (автомасштабирование, устарел — см. docker-autoscaler)
# - kubernetes (для запуска в Kubernetes)
# - custom, ssh, parallels, virtualbox, docker-autoscaler
```

# 3. Настройка конфигурации

Конфигурационный файл находится в `/etc/gitlab-runner/config.toml`. Этот файл создается автоматически после регистрации runner'а.

## Подробный пример конфигурации

```toml
# Максимальное количество одновременно выполняемых задач
concurrent = 3

# Интервал проверки новых задач (в секундах)
check_interval = 0

# Настройки логирования
log_level = "info"
log_format = "runner"

# Конфигурация runner'а
[[runners]]
  # Имя runner'а, которое вы указали при регистрации
  name = "my-runner"
  
  # URL вашего GitLab сервера
  url = "https://gitlab.com/"
  
  # Token, полученный при регистрации
  token = "YOUR-TOKEN"
  
  # Тип executor'а
  executor = "docker"
  
  # Настройки Docker executor'а
  [runners.docker]
    # Базовый образ для запуска задач
    image = "alpine:latest"
    
    # Проверка TLS при подключении к Docker daemon
    tls_verify = false
    
    # Запуск в привилегированном режиме
    privileged = false
    
    # Отключение кэширования
    disable_cache = false
    
    # Монтирование томов
    volumes = [
      "/cache",
      "/var/run/docker.sock:/var/run/docker.sock"
    ]
    
    # Размер shared memory
    shm_size = 0
    
    # Политика перезапуска контейнера
    pull_policy = ["if-not-present"]
    
    # Настройки сети
    network_mode = "host"

  # Настройки кэширования
  [runners.cache]
    Type = "s3"
    Shared = true

  # Настройки SSH
  [runners.ssh]
    user = "gitlab-runner"
    password = "password"
    host = "gitlab.example.com"
    port = "22"
```

> **Примечание:** таймаут задачи не настраивается в `config.toml` — он задаётся ключом `timeout:` в `.gitlab-ci.yml` или в настройках проекта (Settings > CI/CD > Timeout). Блок `[runners.custom]` предназначен только для executor'а `custom` (параметры `config_exec`, `prepare_exec` и т.д.), а не для таймаутов.

# 4. Настройка Executor'ов

## Docker Executor (подробно)

```toml
[[runners]]
  executor = "docker"
  [runners.docker]
    # Базовый образ для запуска задач
    image = "alpine:latest"
    
    # Режим работы с привилегиями
    privileged = false
    
    # Отключение кэширования
    disable_cache = false
    
    # Монтирование томов
    volumes = [
      "/cache",
      "/var/run/docker.sock:/var/run/docker.sock",
      "/builds:/builds:rw"
    ]
    
    # Размер shared memory
    shm_size = 0
    
    # Политика загрузки образов
    pull_policy = ["if-not-present"]
    
    # Настройки сети
    network_mode = "host"
    
    # Лимиты ресурсов
    cpus = "2"
    memory = "2g"
    memory_swap = "2g"
    
    # Настройки DNS
    dns = ["8.8.8.8", "8.8.4.4"]
    
    # Дополнительные переменные окружения
    environment = ["DOCKER_TLS_CERTDIR="]
```

## Shell Executor (подробно)

```toml
[[runners]]
  executor = "shell"
  
  # Настройка оболочки
  shell = "bash"
  
  # Настройки сборки
  builds_dir = "/home/gitlab-runner/builds"
  cache_dir = "/home/gitlab-runner/cache"

  # Переменные окружения
  environment = [
    "LC_ALL=en_US.UTF-8",
    "LANG=en_US.UTF-8",
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
  ]
```

# 5. Управление Runner'ом

## Подробные команды управления

```bash
# Запуск сервиса
sudo systemctl start gitlab-runner
# или
sudo gitlab-runner start

# Остановка сервиса
sudo systemctl stop gitlab-runner
# или
sudo gitlab-runner stop

# Перезапуск сервиса
sudo systemctl restart gitlab-runner
# или
sudo gitlab-runner restart

# Проверка статуса
sudo systemctl status gitlab-runner
# или
sudo gitlab-runner status

# Проверка версии
gitlab-runner --version

# Просмотр списка всех зарегистрированных runner'ов
sudo gitlab-runner list

# Проверка конфигурации
sudo gitlab-runner verify

# Отмена регистрации runner'а
sudo gitlab-runner unregister --name "runner-name"
# или по URL и token
sudo gitlab-runner unregister --url "gitlab-url" --token "token"
```

> Команды `gitlab-runner clean` / `gitlab-runner cache clean` не существуют. Для очистки удалите каталог сборок и кэша вручную (`/home/gitlab-runner/builds`, `/cache`) или настройте lifecycle-политику в объектном хранилище для S3-кэша.

# 6. Обслуживание

## Обновление Runner'а

```bash
# Обновление списка пакетов
sudo apt-get update

# Обновление GitLab Runner
sudo apt-get install gitlab-runner

# Перезапуск сервиса после обновления
sudo systemctl restart gitlab-runner
```

## Очистка и обслуживание

```bash
# Ручная очистка каталога кэша (путь зависит от конфигурации volumes/cache_dir)
sudo rm -rf /cache/*

# Проверка статуса всех зарегистрированных runner'ов
gitlab-runner verify

# Просмотр использования ресурсов
sudo systemctl status gitlab-runner
```

# 7. Рекомендации по безопасности

1. Регулярное обновление:
   * Настройте автоматическое обновление системы
   * Регулярно проверяйте наличие обновлений GitLab Runner
   * Следите за уведомлениями о безопасности от GitLab
2. Настройка HTTPS:
   * Используйте только HTTPS для связи с GitLab
   * Настройте валидные SSL сертификаты
   * Регулярно обновляйте сертификаты
3. Управление доступом:
   * Ограничьте доступ к конфигурационным файлам
   * Используйте отдельного пользователя для runner'а
   * Настройте правильные разрешения на файлы
4. Изоляция и безопасность:
   * Используйте теги для контроля доступа к runner'ам
   * Настройте timeout и ограничения ресурсов
   * Изолируйте runner'ы в отдельных средах
5. Мониторинг:
   * Настройте мониторинг работы runner'а
   * Регулярно проверяйте логи
   * Настройте оповещения о проблемах
6. Резервное копирование:
   * Регулярно сохраняйте резервные копии конфигурации
   * Документируйте все изменения в настройках
   * Храните копии регистрационных токенов в безопасном месте
