# Содержание

## DevOps

* [Docker и Docker Compose](devops/docker-i-docker-compose.md) — руководство по контейнеризации приложений с Docker и оркестрации с помощью Docker Compose.
* [Ansible](devops/ansible.md) — документация по автоматизации задач с использованием Ansible.
* [Terraform](devops/terraform.md) — документация по управлению инфраструктурой как кодом с помощью Terraform.
* [Kubernetes](devops/kubernetes.md) — шпаргалка по kubectl, манифестам и Helm.
* [Управление секретами](devops/upravlenie-sekretami.md) — шифрование секретов с SOPS и хранение в HashiCorp Vault.

## WEB

* [Nginx](web/nginx.md) — высокопроизводительный веб-сервер и обратный прокси, используемый для балансировки нагрузки, кэширования и обработки веб-трафика.
* [Certbot и Let's Encrypt](web/certbot-lets-encrypt.md) — бесплатные TLS-сертификаты и их автообновление.

## Linux

* [Удобный терминал](linux/convenient-terminal.md) — настройка и оптимизация терминала Linux для удобной и продуктивной работы.
* [IPtables](linux/iptables.md) — управление сетевым трафиком и брандмауэром в Linux.
* [nftables](linux/nftables.md) — современная замена iptables: единый синтаксис и постоянная конфигурация.
* [SSH hardening и fail2ban](linux/ssh-hardening-i-fail2ban.md) — безопасная настройка SSH и защита от перебора паролей.
* [systemd](linux/systemd.md) — управление службами, unit-файлы, журналы journalctl и таймеры вместо cron.

## Windows

* [WinGet](windows/winget.md) — современный пакетный менеджер Windows.

## Системы управления версиями (VCS)

* [Шпаргалка Git](vcs/git.md) — справочник по основным командам и особенностям работы с системой контроля версий Git.
* [Установка и настройка GitLab EE](vcs/ustanovka-i-nastroika-gitlab-ee.md)
* [Установка и настройка GitLab Runner](vcs/ustanovka-i-nastroika-gitlab-runner.md)
* [Примеры GitLab CI/CD](vcs/gitlab-ci-cd-primery.md) — типовые пайплайны: сборка, тесты, Docker, деплой, review apps.
* [Обслуживание GitLab EE](vcs/obsluzhivanie-gitlab-ee.md) — резервное копирование, обновление и Container Registry.

## Мониторинг

* [Grafana](monitoring/grafana.md) — инструкция по развёртыванию Grafana.
* [Prometheus](monitoring/prometheus.md) — сбор метрик с node_exporter и основы PromQL.
* [Alertmanager](monitoring/alertmanager.md) — маршрутизация уведомлений в Telegram и email.
* [Loki и Promtail](monitoring/loki-i-promtail.md) — сбор и просмотр логов в Grafana.
* [Uptime Kuma](monitoring/uptime-kuma.md) — простой мониторинг доступности сервисов.

## Базы данных

* [PostgreSQL](database/postgresql.md) — установка, пользователи, резервное копирование и базовый тюнинг.

## Артефакты

* [Nexus Sonatype](artifacts/nexus-sonatype.md) — инструмент для управления репозиториями, используемый для хранения, проксирования и управления артефактами (библиотеки, зависимости, Docker-образы и т.д.) в процессе разработки ПО.
