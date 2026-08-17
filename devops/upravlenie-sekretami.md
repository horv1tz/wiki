---
description: Управление секретами — SOPS и HashiCorp Vault
---

# Управление секретами

Пароли, ключи и токены не должны храниться в репозитории в открытом виде. Два популярных подхода:

* **SOPS** — шифрование файлов (YAML/JSON/ENV) прямо в репозитории; ключи — age, PGP, KMS
* **HashiCorp Vault** — централизованное хранилище секретов с контролем доступа и аудитом

## SOPS

### 1. Установка SOPS и age

```bash
# age — современный инструмент шифрования (ключи для SOPS)
sudo apt install age

# SOPS — бинарник с GitHub
wget https://github.com/getsops/sops/releases/download/v3.9.1/sops-v3.9.1.linux.amd64
sudo mv sops-v3.9.1.linux.amd64 /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops

# Генерация пары ключей age
age-keygen -o ~/.config/sops/age/keys.txt
# Публичный ключ будет выведен: age1...
```

### 2. Конфигурация проекта

Файл `.sops.yaml` в корне репозитория:

```yaml
creation_rules:
  - path_regex: secrets/.*\.yaml$
    age: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrkh2axpvs5gcsvu8gr   # публичный ключ
```

### 3. Работа с секретами

```bash
# Создать/отредактировать зашифрованный файл (в репозитории лежит шифротекст)
sops secrets/prod.yaml

# Посмотреть расшифрованное содержимое
sops -d secrets/prod.yaml

# Выполнить команду с секретами из env-файла
sops exec-file secrets/prod.env './deploy.sh {}'
```

### 4. SOPS в CI/CD

Приватный ключ age передаётся в пайплайн через защищённую переменную (`SOPS_AGE_KEY`), после чего скрипты расшифровывают файлы на лету:

```yaml
deploy:
  script:
    - export SOPS_AGE_KEY="$SOPS_AGE_KEY"
    - sops -d secrets/prod.env > /tmp/prod.env
    - source /tmp/prod.env && ./deploy.sh
```

## HashiCorp Vault

### 1. Установка и запуск (dev-режим для знакомства)

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault

# Dev-режим (только для экспериментов, данные в памяти)
vault server -dev
```

```bash
export VAULT_ADDR='http://127.0.0.1:8200'

# Записать секрет
vault kv put secret/myapp db_password="s3cret"

# Прочитать
vault kv get secret/myapp
vault kv get -field=db_password secret/myapp
```

### 2. Ключевые возможности Vault (для продакшена)

* **Политики доступа** — кто к каким секретам имеет доступ
* **Аудит** — журнал всех обращений
* **Динамические секреты** — временные учётные данные для БД, облаков
* **Аренда и ротация** — автоматическая смена секретов

### 3. Vault в GitLab CI

GitLab умеет получать секреты из Vault нативно через JWT-аутентификацию:

```yaml
job:
  secrets:
    DB_PASSWORD:
      vault: myapp/db_password@kv   # путь/поле@движок
      file: false
```

## Что выбрать

| Задача | Инструмент |
| --- | --- |
| Секреты для небольшого проекта в Git | SOPS |
| Много команд, окружений, нужен аудит | Vault |
| Секреты только для CI | GitLab CI/CD Variables |

## Полезные ссылки

* [SOPS](https://github.com/getsops/sops) и [age](https://github.com/FiloSottile/age)
* [Документация Vault](https://developer.hashicorp.com/vault/docs)
* [Vault в GitLab CI](https://docs.gitlab.com/ee/ci/secrets/)
