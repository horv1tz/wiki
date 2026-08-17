---
description: Бэкапы, обновление и Container Registry GitLab EE
---

# Обслуживание GitLab EE

Продолжение статьи [Установка и настройка GitLab EE](ustanovka-i-nastroika-gitlab-ee.md): резервное копирование, обновление и встроенный реестр контейнеров.

## 1. Резервное копирование

### 1.1 Ручной бэкап

```bash
sudo gitlab-backup create
```

Бэкап сохраняется в `/var/opt/gitlab/backups/` в формате `<timestamp>_gitlab_backup.tar`.

### 1.2 Что НЕ входит в бэкап

Скопируйте отдельно — без них восстановление будет неполным:

```bash
sudo cp /etc/gitlab/gitlab.rb /backup-location/
sudo cp -r /etc/gitlab/gitlab-secrets.json /backup-location/
```

> `gitlab-secrets.json` критичен: без него невозможно расшифровать данные из бэкапа (токены, переменные CI, двухфакторная аутентификация).

### 1.3 Автоматический бэкап (cron для root)

```cron
# Каждый день в 02:00, отправка в S3 настраивается в gitlab.rb
0 2 * * * /usr/bin/gitlab-backup create CRON=1
```

`CRON=1` подавляет прогресс-вывод, чтобы cron не присылал письма.

### 1.4 Настройка выгрузки в S3 (`/etc/gitlab/gitlab.rb`)

```ruby
gitlab_rails['backup_upload_connection'] = {
  'provider' => 'AWS',
  'region' => 'eu-central-1',
  'aws_access_key_id' => '<ключ>',
  'aws_secret_access_key' => '<секрет>'
}
gitlab_rails['backup_upload_remote_directory'] = 'gitlab-backups'
gitlab_rails['backup_keep_time'] = 604800   # хранить локально 7 дней (в секундах)
```

### 1.5 Восстановление

```bash
# Версия восстанавливаемого GitLab должна совпадать с версией бэкапа
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-ctl status   # убедиться, что остановлены

sudo gitlab-backup restore BACKUP=<timestamp из имени файла>

sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true
```

## 2. Обновление

```bash
sudo apt update
sudo apt install gitlab-ee
```

* Соблюдайте **upgrade path** — между мажорными версиями есть обязательные промежуточные релизы: [GitLab Upgrade Path Tool](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/)
* Перед обновлением снимите свежий бэкап
* Фоновые миграции должны завершиться до следующего обновления: **Admin > Monitoring > Background Migrations**

## 3. Container Registry

Встроенный реестр Docker-образов для проектов.

### 3.1 Включение (`/etc/gitlab/gitlab.rb`)

```ruby
registry_external_url 'https://registry.example.com'

# Или на том же домене, другом порту:
# registry['registry_http_addr'] = "0.0.0.0:5050"
```

```bash
sudo gitlab-ctl reconfigure
```

Проверка:

```bash
docker login registry.example.com
docker tag myapp:latest registry.example.com/group/project:latest
docker push registry.example.com/group/project:latest
```

### 3.2 Очистка реестра

Реестр сам не удаляет старые образы — включите политики очистки на уровне проекта (**Settings > Packages and registries > Cleanup policies**) или очистку тегов через API:

```bash
# Удалить теги старше 30 дней, оставив 5 последних
curl --request DELETE --header "PRIVATE-TOKEN: <токен>" \
  "https://gitlab.example.com/api/v4/projects/<id>/registry/repositories/<repo_id>/tags?name_regex_delete=.*&keep_n=5&older_than=30d"
```

### 3.3 Garbage collection (освобождение диска)

```bash
sudo gitlab-ctl registry-garbage-collect -m   # -m: удалить неиспользуемые манифесты
```

## 4. Мониторинг состояния

```bash
sudo gitlab-rake gitlab:check SANITIZE=true   # самодиагностика
sudo gitlab-ctl status                        # статус сервисов
sudo gitlab-ctl tail                          # логи всех сервисов
sudo gitlab-ctl tail nginx                    # логи конкретного сервиса
```

## 5. Полезные ссылки

* [Backup and restore](https://docs.gitlab.com/ee/administration/backup_restore/)
* [Upgrade paths](https://docs.gitlab.com/ee/update/upgrade_paths.html)
* [Container Registry administration](https://docs.gitlab.com/ee/administration/packages/container_registry.html)
