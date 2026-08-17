---
description: Установка и базовое администрирование PostgreSQL — пользователи, бэкапы, тюнинг
---

# PostgreSQL

**PostgreSQL** — мощная объектно-реляционная СУБД с открытым исходным кодом. Здесь — установка, базовое администрирование и резервное копирование.

## 1. Установка

```bash
# Debian/Ubuntu
sudo apt install postgresql

# Или свежая версия из официального репозитория PostgreSQL (PGDG)
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update && sudo apt install postgresql-16

sudo systemctl enable --now postgresql
```

## 2. Первые шаги

По умолчанию локальный вход идёт через системного пользователя `postgres` (аутентификация peer):

```bash
sudo -u postgres psql
```

```sql
-- Создать пользователя и базу
CREATE USER appuser WITH PASSWORD 'strong-password';
CREATE DATABASE appdb OWNER appuser;

-- Права
GRANT ALL PRIVILEGES ON DATABASE appdb TO appuser;

\l          -- список баз
\du         -- список ролей
\dt         -- таблицы текущей базы
\q          -- выход
```

То же самое из командной строки:

```bash
sudo -u postgres createuser appuser --pwprompt
sudo -u postgres createdb appdb -O appuser
```

## 3. Доступ по сети (при необходимости)

`/etc/postgresql/16/main/postgresql.conf`:

```
listen_addresses = 'localhost, 10.0.0.5'
```

`/etc/postgresql/16/main/pg_hba.conf` — кто и как может подключаться:

```
# TYPE  DATABASE  USER  ADDRESS          METHOD
host    appdb     appuser  10.0.0.0/24    scram-sha-256
```

```bash
sudo systemctl reload postgresql
```

> Открывайте доступ только для нужных подсетей и обязательно ограничивайте порт 5432 брандмауэром.

## 4. Резервное копирование

```bash
# Дамп одной базы (сжатый формат)
sudo -u postgres pg_dump -Fc appdb > /backups/appdb-$(date +%F).dump

# Дамп всех баз и глобальных объектов (роли, табличные пространства)
sudo -u postgres pg_dumpall > /backups/all-$(date +%F).sql

# Восстановление
sudo -u postgres pg_restore -d appdb /backups/appdb-2026-08-17.dump
sudo -u postgres psql -f /backups/all-2026-08-17.sql
```

Пример задания для systemd-таймера или cron (ежедневно в 3:00, хранить 14 дней):

```bash
#!/bin/bash
# /opt/scripts/pg-backup.sh
BACKUP_DIR=/backups/postgresql
mkdir -p "$BACKUP_DIR"
sudo -u postgres pg_dump -Fc appdb > "$BACKUP_DIR/appdb-$(date +%F).dump"
find "$BACKUP_DIR" -name "*.dump" -mtime +14 -delete
```

## 5. Мониторинг и обслуживание

```sql
-- Активные запросы
SELECT pid, usename, state, query FROM pg_stat_activity;

-- Размеры баз
SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;
```

```bash
# Логи
sudo journalctl -u postgresql -f
```

PostgreSQL сам выполняет автоочистку (autovacuum); вручную при больших обновлениях:

```sql
VACUUM ANALYZE;
```

## 6. Минимальный тюнинг (postgresql.conf)

Ориентиры для сервера с 8 ГБ RAM — точные значения подбираются под нагрузку:

```
shared_buffers = 2GB                # ~25% RAM
effective_cache_size = 6GB          # ~75% RAM (подсказка планировщику)
work_mem = 32MB                     # память на сортировку в запросе
maintenance_work_mem = 512MB
max_connections = 100
```

Онлайн-калькулятор: [pgtune.leopard.in.ua](https://pgtune.leopard.in.ua/). После изменения `shared_buffers` нужен `restart`, для большинства остальных — `reload`.

## 7. Полезные ссылки

* [Документация PostgreSQL](https://www.postgresql.org/docs/)
* [Репозиторий PGDG](https://www.postgresql.org/download/linux/ubuntu/)
* Экспортер метрик для Prometheus: [postgres_exporter](https://github.com/prometheus-community/postgres_exporter)
