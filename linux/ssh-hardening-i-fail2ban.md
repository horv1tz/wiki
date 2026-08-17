---
description: Безопасная настройка SSH и защита от брутфорса с fail2ban
---

# SSH hardening и fail2ban

SSH — главная точка входа на сервер и самая частая цель атак. Минимальный набор мер: ключи вместо паролей, запрет root-входа, fail2ban против перебора.

## 1. Вход по ключам

```bash
# На локальной машине — сгенерировать ключ
ssh-keygen -t ed25519 -C "email@example.com"

# Скопировать публичный ключ на сервер
ssh-copy-id user@<сервер>
```

## 2. Настройка sshd

Отредактируйте `/etc/ssh/sshd_config` (или создайте файл с переопределениями в `/etc/ssh/sshd_config.d/hardening.conf` — предпочтительный современный способ):

```
# Только ключи, пароли отключены
PasswordAuthentication no
PubkeyAuthentication yes

# Запрет прямого входа root
PermitRootLogin no

# Только конкретные пользователи
AllowUsers deploy admin

# Отключить лишнее
X11Forwarding no
PermitEmptyPasswords no

# (Опционально) нестандартный порт — уменьшает шум от ботов, НЕ заменяет ключи
# Port 2222
```

Проверка конфигурации и перезапуск:

```bash
sudo sshd -t                      # проверка синтаксиса
sudo systemctl reload ssh
```

> **Важно:** перед отключением паролей убедитесь, что вход по ключу работает — проверьте в новой сессии, не закрывая текущую.

## 3. fail2ban — защита от перебора

### 3.1 Установка

```bash
sudo apt install fail2ban
```

### 3.2 Конфигурация `/etc/fail2ban/jail.local`

Не редактируйте `jail.conf` — он перезаписывается при обновлениях:

```ini
[DEFAULT]
# Время блокировки
bantime = 1h
# Окно подсчёта попыток и их лимит
findtime = 10m
maxretry = 5
# Не блокировать свои адреса
ignoreip = 127.0.0.1/8 192.168.1.0/24

[sshd]
enabled = true
port = ssh
# если SSH на нестандартном порту:
# port = 2222
backend = systemd
```

```bash
sudo systemctl enable --now fail2ban
```

### 3.3 Управление

```bash
# Статус и список активных jail'ов
sudo fail2ban-client status

# Заблокированные IP в jail sshd
sudo fail2ban-client status sshd

# Разблокировать IP
sudo fail2ban-client unban 203.0.113.50

# Логи
sudo journalctl -u fail2ban -f
```

### 3.4 Дополнительные jail'ы

```ini
[nginx-http-auth]
enabled = true

[nginx-botsearch]
enabled = true

[recidive]
enabled = true    # длительная блокировка рецидивистов
bantime = 1w
```

## 4. Дополнительные меры

* **2FA для SSH**: модуль `pam_google_authenticator` — одноразовые коды поверх ключей
* **knockd / port knocking**: SSH-порт скрыт до специальной последовательности подключений
* **VPN-first**: SSH доступен только из VPN (WireGuard) — порт закрыт для интернета брандмауэром
* **Автообновления безопасности**: `sudo apt install unattended-upgrades`

## 5. Полезные ссылки

* [Документация OpenSSH](https://www.openssh.com/manual.html)
* [fail2ban](https://github.com/fail2ban/fail2ban)
