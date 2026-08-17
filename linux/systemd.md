---
description: Управление службами systemd — unit-файлы, journalctl, таймеры
---

# systemd

**systemd** — система инициализации и менеджер служб в большинстве современных дистрибутивов Linux. Управляет сервисами, логами, точками монтирования, таймерами и загрузкой системы.

## 1. Управление службами (systemctl)

```bash
systemctl status <сервис>          # состояние и последние логи
sudo systemctl start <сервис>      # запуск
sudo systemctl stop <сервис>       # остановка
sudo systemctl restart <сервис>    # перезапуск
sudo systemctl reload <сервис>     # перечитать конфигурацию без разрыва соединений

sudo systemctl enable <сервис>     # автозапуск при загрузке
sudo systemctl disable <сервис>    # убрать из автозапуска
sudo systemctl enable --now <сервис>   # включить и сразу запустить

systemctl list-units --type=service --state=running   # запущенные сервисы
systemctl list-unit-files | grep enabled              # все сервисы в автозапуске
systemctl --failed                   # упавшие юниты
systemctl daemon-reload              # перечитать unit-файлы (после их изменения)
```

## 2. Написание unit-файла

Пример сервиса `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=My Application
Documentation=https://example.com/docs
After=network-online.target postgresql.service
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server --config /etc/myapp/config.yaml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

# Переменные окружения
Environment=APP_ENV=production
EnvironmentFile=-/etc/myapp/env      # '-' = файл не обязателен

# Безопасность
NoNewPrivileges=true
ProtectSystem=strict
ReadWritePaths=/var/lib/myapp

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

Ключевые директивы:

* `Type=` — `simple` (процесс не форкается), `forking` (классический демон), `oneshot` (разовая задача), `notify`
* `Restart=` — `no`, `on-failure`, `always`; `RestartSec=` — пауза перед перезапуском
* `After=` / `Wants=` / `Requires=` — порядок и зависимости запуска

## 3. Логи (journalctl)

```bash
journalctl -u <сервис>             # логи сервиса
journalctl -u <сервис> -f          # в реальном времени
journalctl -u <сервис> --since today
journalctl -u <сервис> -p err      # только ошибки
journalctl -b                      # логи текущей загрузки
journalctl -b -1                   # логи предыдущей загрузки (диагностика падений)
journalctl --disk-usage            # сколько места занимают логи

# Ограничение размера журнала: /etc/systemd/journald.conf
# SystemMaxUse=500M
```

## 4. Таймеры вместо cron

Таймеры прозрачнее cron: видны в `systemctl list-timers`, имеют логи и зависимости.

Сервис `/etc/systemd/system/backup.service`:

```ini
[Unit]
Description=Backup

[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.sh
```

Таймер `/etc/systemd/system/backup.timer`:

```ini
[Unit]
Description=Daily backup

[Timer]
OnCalendar=*-*-* 03:30:00      # каждый день в 03:30
Persistent=true                # выполнить пропущенный запуск после простоя

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

systemctl list-timers                   # все таймеры и время следующего запуска
systemd-analyze calendar "Mon 03:00"    # проверить расписание
```

Формат `OnCalendar`: `DayOfWeek Year-Month-Day Hour:Minute:Second`, например `Mon *-*-* 03:00:00` (по понедельникам в 3:00), `hourly`, `daily`, `weekly`.

## 5. Диагностика загрузки и производительности

```bash
systemd-analyze                  # сколько заняла загрузка системы
systemd-analyze blame            # какие юниты дольше всего стартовали
systemd-analyze critical-chain   # критическая цепочка зависимостей
```

## 6. Полезные ссылки

* [systemd.unit(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html) — справочник по директивам
* [systemd.timer(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html) — синтаксис OnCalendar
