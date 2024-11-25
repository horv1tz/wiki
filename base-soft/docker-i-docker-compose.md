---
description: Подробная Шпаргалка по Docker
---

# Docker и Docker Compose

Docker — это платформа для разработки, доставки и запуска приложений в контейнерах. Контейнеры позволяют упаковать приложение с его зависимостями и запускать его в изолированной среде на любой системе с установленным Docker.

### Основные Понятия

* **Образы (Images):** Шаблоны для создания контейнеров. Образы состоят из слоев, каждый из которых представляет собой набор файловых изменений.
* **Контейнеры (Containers):** Изолированные экземпляры образов, которые можно запускать, останавливать и управлять ими.
* **Dockerfile:** Файл с инструкциями для создания пользовательских образов.
* **Docker Hub:** Публичный реестр образов Docker, где можно найти и загрузить готовые образы.

### Установка Docker

#### На Ubuntu

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

#### На MacOS

Скачайте Docker Desktop с официального сайта и установите его.

#### Проверка установки

```bash
docker --version
```

### Основные Команды Docker

#### Работа с Образами

*   **Поиск образов в Docker Hub:**

    ```bash
    docker search <имя_образа>
    ```

    **Пример:**

    ```bash
    docker search nginx
    ```
*   **Загрузка образа:**

    ```bash
    docker pull <имя_образа>
    ```

    **Пример:**

    ```bash
    docker pull nginx
    ```
*   **Просмотр локальных образов:**

    ```bash
    docker images
    ```
*   **Удаление образа:**

    ```bash
    docker rmi <ID_образа>
    ```

#### Работа с Контейнерами

*   **Запуск контейнера:**

    ```bash
    docker run <опции> <имя_образа>
    ```

    **Пример (запуск Nginx на порту 80):**

    ```bash
    docker run -d -p 80:80 nginx
    ```
* **Опции команды `docker run`:**
  * `-d`: Запуск в фоновом режиме (detached mode).
  * `-p`: Проброс порта (HOST:CONTAINER).
  * `--name`: Присвоение имени контейнеру.
  * `-v`: Монтирование тома (HOST:CONTAINER).
*   **Просмотр запущенных контейнеров:**

    ```bash
    docker ps
    ```
*   **Просмотр всех контейнеров (включая остановленные):**

    ```bash
    docker ps -a
    ```
*   **Остановка контейнера:**

    ```bash
    docker stop <ID_контейнера>
    ```
*   **Запуск остановленного контейнера:**

    ```bash
    docker start <ID_контейнера>
    ```
*   **Перезапуск контейнера:**

    ```bash
    docker restart <ID_контейнера>
    ```
*   **Удаление контейнера:**

    ```bash
    docker rm <ID_контейнера>
    ```
*   **Удаление всех остановленных контейнеров:**

    ```bash
    docker container prune
    ```

#### Выполнение команд внутри контейнера

*   **Выполнение команды в запущенном контейнере:**

    ```bash
    docker exec -it <ID_контейнера> <команда>
    ```

    **Пример (открыть bash):**

    ```bash
    docker exec -it <ID_контейнера> /bin/bash
    ```
*   **Запуск контейнера с доступом к терминалу:**

    ```bash
    docker run -it <имя_образа> /bin/bash
    ```

#### Логи и мониторинг

*   **Просмотр логов контейнера:**

    ```bash
    docker logs <ID_контейнера>
    ```

    **С выводом в реальном времени:**

    ```bash
    docker logs -f <ID_контейнера>
    ```
*   **Статистика использования ресурсов:**

    ```bash
    docker stats
    ```

#### Управление сетью

*   **Просмотр сетей Docker:**

    ```bash
    docker network ls
    ```
*   **Создание пользовательской сети:**

    ```bash
    docker network create <имя_сети>
    ```
*   **Запуск контейнера в определенной сети:**

    ```bash
    docker run --network=<имя_сети> <имя_образа>
    ```
*   **Подключение контейнера к сети:**

    ```bash
    docker network connect <имя_сети> <ID_контейнера>
    ```
*   **Отключение контейнера от сети:**

    ```bash
    docker network disconnect <имя_сети> <ID_контейнера>
    ```

#### Работа с томами

*   **Просмотр томов:**

    ```bash
    docker volume ls
    ```
*   **Создание тома:**

    ```bash
    docker volume create <имя_тома>
    ```
*   **Удаление тома:**

    ```bash
    docker volume rm <имя_тома>
    ```
*   **Монтирование тома при запуске контейнера:**

    ```bash
    docker run -v <имя_тома>:/path/in/container <имя_образа>
    ```

    **Пример (монтирование текущей директории):**

    ```bash
    docker run -v $(pwd):/app <имя_образа>
    ```

### Создание Собственного Образа с помощью Dockerfile

#### Структура Dockerfile

```Dockerfile
# Базовый образ
FROM <базовый_образ>

# Автор
LABEL maintainer="ваш_email@example.com"

# Установка зависимостей
RUN apt-get update && apt-get install -y <пакеты>

# Копирование файлов
COPY <источник> <назначение>

# Установка рабочей директории
WORKDIR <путь>

# Определение переменных окружения
ENV <переменная>=<значение>

# Открытие портов
EXPOSE <порты>

# Команда запуска
CMD ["команда", "аргумент"]
```

#### Пример Dockerfile для Python приложения

```Dockerfile
# Базовый образ
FROM python:3.9-slim

# Установка рабочей директории
WORKDIR /app

# Копирование зависимостей
COPY requirements.txt .

# Установка зависимостей
RUN pip install -r requirements.txt

# Копирование исходного кода
COPY . .

# Открытие порта
EXPOSE 5000

# Команда запуска
CMD ["python", "app.py"]
```

#### Сборка образа

```bash
docker build -t <имя_образа>:<тег> .
```

**Пример:**

```bash
docker build -t myapp:latest .
```

#### Запуск созданного образа

```bash
docker run -d -p 5000:5000 myapp:latest
```

### Docker Compose

Docker Compose используется для определения и запуска многоконтейнерных приложений.

#### Файл docker-compose.yml

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    links:
      - redis
  redis:
    image: "redis:alpine"
```

#### Основные команды Docker Compose

*   **Запуск всех сервисов:**

    ```bash
    docker-compose up
    ```

    **В фоновом режиме:**

    ```bash
    docker-compose up -d
    ```
*   **Остановка всех сервисов:**

    ```bash
    docker-compose down
    ```
*   **Пересборка образов:**

    ```bash
    docker-compose build
    ```
*   **Просмотр логов:**

    ```bash
    docker-compose logs
    ```
*   **Выполнение команды внутри сервиса:**

    ```bash
    docker-compose exec <сервис> <команда>
    ```

    **Пример:**

    ```bash
    docker-compose exec web /bin/bash
    ```

### Docker Registry

*   **Вход в Docker Hub:**

    ```bash
    docker login
    ```
*   **Тегирование образа для отправки в реестр:**

    ```bash
    docker tag <локальный_образ> <репозиторий>/<имя_образа>:<тег>
    ```

    **Пример:**

    ```bash
    docker tag myapp:latest username/myapp:latest
    ```
*   **Отправка образа в Docker Hub:**

    ```bash
    docker push <репозиторий>/<имя_образа>:<тег>
    ```
*   **Получение образа из реестра:**

    ```bash
    docker pull <репозиторий>/<имя_образа>:<тег>
    ```

### Docker Networking

#### Типы сетей

* **bridge:** По умолчанию для контейнеров на одном хосте.
* **host:** Контейнер использует сеть хоста.
* **none:** Отключенная сеть.

#### Создание пользовательской мостовой сети

```bash
docker network create mynetwork
```

#### Запуск контейнера в пользовательской сети

```bash
docker run --network=mynetwork <имя_образа>
```

#### Связывание контейнеров

Контейнеры в одной сети могут обращаться друг к другу по имени.

**Пример:**

*   Запустить Redis:

    ```bash
    docker run -d --name redis --network=mynetwork redis
    ```
*   Запустить приложение, которое использует Redis:

    ```bash
    docker run -d --network=mynetwork myapp
    ```

В приложении можно обращаться к Redis по имени `redis`.

### Docker Volumes (Тома)

Тома используются для сохранения данных контейнера вне его файловой системы.

#### Создание тома

```bash
docker volume create myvolume
```

#### Использование тома при запуске контейнера

```bash
docker run -d -v myvolume:/data myapp
```

#### Монтирование директории хоста

```bash
docker run -d -v /host/path:/container/path myapp
```

#### Резервное копирование тома

```bash
docker run --rm -v myvolume:/volume -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz /volume
```

#### Восстановление тома из резервной копии

```bash
docker run --rm -v myvolume:/volume -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /
```

### Управление ресурсами

#### Ограничение использования CPU

```bash
docker run --cpus="1.5" <имя_образа>
```

#### Ограничение использования памяти

```bash
docker run -m 512m <имя_образа>
```

### Полезные Команды

*   **Удаление всех остановленных контейнеров:**

    ```bash
    docker container prune
    ```
*   **Удаление всех неиспользуемых образов:**

    ```bash
    docker image prune
    ```
*   **Удаление всех неиспользуемых данных:**

    ```bash
    docker system prune
    ```
*   **Информация о Docker:**

    ```bash
    docker info
    ```
*   **Список процессов внутри контейнера:**

    ```bash
    docker top <ID_контейнера>
    ```

### Docker Swarm (Оркестрация)

Docker Swarm позволяет управлять кластером Docker-демонов.

#### Инициализация Swarm

```bash
docker swarm init
```

#### Добавление узла в Swarm

На основном узле:

```bash
docker swarm join-token worker
```

Следуйте инструкциям для добавления узла.

#### Развертывание сервиса в Swarm

```bash
docker service create --name myservice -p 80:80 nginx
```

#### Масштабирование сервиса

```bash
docker service scale myservice=5
```

#### Просмотр сервисов

```bash
docker service ls
```

### Безопасность Docker

* **Запуск только доверенных образов.**
* **Регулярное обновление образов и Docker Engine.**
* **Использование минимальных базовых образов (например, Alpine).**
*   **Ограничение привилегий контейнеров:**

    ```bash
    docker run --user nobody <имя_образа>
    ```
*   **Использование `--read-only` для создания только чтения файловой системы:**

    ```bash
    docker run --read-only <имя_образа>
    ```

### Docker Best Practices

* **Минимизируйте количество слоев в Dockerfile.**
* **Используйте `.dockerignore` для исключения ненужных файлов.**
* **Закрепляйте версии зависимостей.**
* **Регулярно обновляйте образы.**
* **Используйте многоступенчатую сборку (multi-stage builds) для уменьшения размера образа.**

**Пример многоступенчатой сборки:**

```Dockerfile
# Ступень 1: Сборка приложения
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Ступень 2: Создание конечного образа
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

### Получение Справки

*   **Общая справка:**

    ```bash
    docker --help
    ```
*   **Справка по конкретной команде:**

    ```bash
    docker <команда> --help
    ```

    **Пример:**

    ```bash
    docker run --help
    ```

### Полезные Ресурсы

* **Официальная документация Docker:**\
  https://docs.docker.com/
* **Docker Hub:**\
  https://hub.docker.com/
* **Play with Docker (песочница):**\
  https://labs.play-with-docker.com/

### Заключение

Docker является мощным инструментом для контейнеризации приложений, что упрощает их развертывание и масштабирование. Эта шпаргалка призвана помочь вам быстро найти нужные команды и лучшие практики при работе с Docker. Рекомендуется регулярно изучать официальную документацию и практиковаться для более глубокого понимания возможностей Docker.
