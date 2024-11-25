---
description: Поднятие базового Gitlab EE (Enterprise Edition)
---

# Установка и настройка Gitlab EE

## Процесс установки на Debian/Ubuntu

1. Установите необходимые зависимости:

```bash
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
```

2. Добавьте репозиторий GitLab:

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

3. Установите GitLab EE:

```bash
sudo EXTERNAL_URL="http://gitlab.example.com" apt-get install gitlab-ee
```

Замените gitlab.example.com на ваш домен или IP-адрес.

Для установки конкретной версии:

```bash
sudo EXTERNAL_URL="http://gitlab.example.com" apt-get install gitlab-ee=<версия>
```

4. После установки дождитесь окончания автоматической настройки. Если нужно запустить настройку вручную:

```bash
sudo gitlab-ctl reconfigure
```

5. Для получения начального пароля root пользователя:

```bash
sudo cat /etc/gitlab/initial_root_password
```

Важно: Этот пароль будет удален через 24 часа после установки.

6. Проверьте статус всех сервисов:

```bash
sudo gitlab-ctl status
```

## Изменение конфигурации

Основной файл конфигурации **/etc/gitlab/gitlab.rb** Команда для применения настроек **`sudo gitlab-ctl reconfigure`**

## Настройка URL

Для доступа не только до домену, но и IP необходимо в файле

```
external_url 'http://0.0.0.0'
```

Для доступа по конкретному домену

```
external_url 'http://<домен>'
```

## Настройка DNS

При отсутствии доступа по домену, необходимо прописать в файле /etc/hosts IP-адрес и домен:

```
<IP-адрес>    <домен>
```

## Настройка Timezone

В файле конфигурации надо заменить

```
gitlab_rails['time_zone'] = 'UTC'
```

на

```
gitlab_rails['time_zone'] = 'Europe/Moscow'
```

## Настройка почты

```
gitlab_rails['gitlab_email_enabled'] = true    
gitlab_rails['gitlab_email_from'] = 'noreply@<домен>'  
gitlab_rails['gitlab_email_display_name'] = 'Gitlab Noreply'
```

## Настройка SSH

```
gitlab_rails['gitlab_ssh_host'] = '<домен>'  
gitlab_rails['gitlab_ssh_user'] = 'git'
```

## Настройка LDAP

```
gitlab_rails['ldap_enabled'] = true  
gitlab_rails['prevent_ldap_sign_in'] = false  

###! **remember to close this block with 'EOS' below**  
gitlab_rails['ldap_servers'] = YAML.load <<-EOS #  
main: # 'main' is the GitLab 'provider ID' of this LDAP server  
 label: '<название организации>'  
 host: '<ldap-сервер>'  
 port: 636  
 uid: 'uid'  
 encryption: 'simple_tls' # "start_tls" or "simple_tls" or "plain"  
 verify_certificates: false  
 bind_dn: '<учетная запись для подключения>'  
 password: '<пароль>'  
 active_directory: false  
 allow_username_or_email_login: true  
 lowercase_usernames: false  
 block_auto_created_users: false  
 base: '<база поиска пользователей>'  
 user_filter: '<фильтр пользователей>'  
 attributes:  
   username: ['uid', 'userid', 'sAMAccountName']  
   email:    ['mail', 'email', 'userPrincipalName']  
EOS
```
