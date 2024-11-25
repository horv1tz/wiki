---
description: Подробная инструкция по установке и настройке GitLab Runner для Debian/Ubuntu
---

# Установка и настройка Gitlab Runner

### 1. Установка GitLab Runner

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

### 2. Регистрация Runner

#### Подготовка к регистрации

Перед регистрацией runner'а необходимо получить регистрационные данные:

1. Войдите в веб-интерфейс GitLab
2. Перейдите в Settings > CI/CD > Runners
3. В разделе "Set up a specific Runner manually" вы найдете:
   * URL вашего GitLab instance
   * Registration token

#### Процесс регистрации

Запустите процесс регистрации:

```bash
sudo gitlab-runner register
```

Вам будет предложено ввести следующую информацию:

```bash
# 1. GitLab instance URL
# Введите URL вашего GitLab сервера
# Например: https://gitlab.com/ или https://gitlab.your-domain.com/

# 2. Registration token
# Скопируйте token из веб-интерфейса GitLab

# 3. Описание runner'а
# Введите понятное описание, например:
# "production-server-01" или "docker-runner-staging"

# 4. Теги (опционально)
# Теги помогают организовать и фильтровать runner'ы
# Например: docker,aws,production
# Можно оставить пустым и нажать Enter

# 5. Maintenance note (опционально)
# Дополнительная заметка для обслуживания
# Можно оставить пустым и нажать Enter

# 6. Выбор executor'а
# Выберите один из следующих:
# - shell (выполняет команды напрямую в системе)
# - docker (запускает задачи в Docker контейнерах)
# - docker-windows (для Windows контейнеров)
# - docker+machine (для автоматического масштабирования)
# - kubernetes (для запуска в Kubernetes)
# - custom (пользовательская конфигурация)
# - ssh (для удаленного выполнения)
# - parallels (для Parallels виртуализации)
# - virtualbox (для VirtualBox)
# - docker-autoscaler (для автоматического масштабирования Docker)
```

### 3. Настройка конфигурации

Конфигурационный файл находится в `/etc/gitlab-runner/config.toml`. Этот файл создается автоматически после регистрации runner'а.

#### Подробный пример конфигурации

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
    pull_policy = "if-not-present"
    
    # Настройки сети
    network_mode = "host"

  # Настройки кэширования
  [runners.cache]
    Type = "s3"
    Shared = true
    
  # Таймауты
  [runners.custom]
    build_timeout = 3600    # 1 час
    
  # Настройки SSH
  [runners.ssh]
    user = "gitlab-runner"
    password = "password"
    host = "gitlab.example.com"
    port = "22"
```

### 4. Настройка Executor'ов

#### Docker Executor (подробно)

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
    pull_policy = "if-not-present"
    
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

#### Shell Executor (подробно)

```toml
[[runners]]
  executor = "shell"
  
  # Настройка оболочки
  shell = "bash"
  
  # Настройки сборки
  builds_dir = "/home/gitlab-runner/builds"
  cache_dir = "/home/gitlab-runner/cache"
  
  # Таймауты
  build_timeout = 3600    # 1 час
  
  # Переменные окружения
  environment = [
    "LC_ALL=en_US.UTF-8",
    "LANG=en_US.UTF-8",
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
  ]
```

### 5. Управление Runner'ом

#### Подробные команды управления

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

# Очистка старых сборок
sudo gitlab-runner clean
```

### 6. Обслуживание

#### Обновление Runner'а

```bash
# Обновление списка пакетов
sudo apt-get update

# Обновление GitLab Runner
sudo apt-get install gitlab-runner

# Перезапуск сервиса после обновления
sudo systemctl restart gitlab-runner
```

#### Очистка и обслуживание

```bash
# Очистка кэша
gitlab-runner cache clean

# Очистка старых сборок
gitlab-runner clean

# Проверка статуса всех зарегистрированных runner'ов
gitlab-runner verify

# Просмотр использования ресурсов
sudo systemctl status gitlab-runner
```

### 7. Рекомендации по безопасности

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
