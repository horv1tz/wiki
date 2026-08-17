---
description: Примеры пайплайнов GitLab CI/CD — сборка, тесты, деплой
---

# Примеры GitLab CI/CD

Пайплайн описывается в файле `.gitlab-ci.yml` в корне репозитория и выполняется на [GitLab Runner](ustanovka-i-nastroika-gitlab-runner.md). Здесь собраны типовые конфигурации.

## 1. Базовый пайплайн: build → test → deploy

```yaml
stages:
  - build
  - test
  - deploy

default:
  image: alpine:3.20

build:
  stage: build
  script:
    - echo "Сборка приложения"
  artifacts:
    paths:
      - dist/
    expire_in: 1 day

test:
  stage: test
  script:
    - echo "Запуск тестов"

deploy:
  stage: deploy
  script:
    - echo "Деплой на сервер"
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual        # ручной запуск из интерфейса
```

## 2. Сборка Docker-образа и push в Container Registry

```yaml
build-image:
  stage: build
  image: docker:27
  services:
    - docker:27-dind
  variables:
    DOCKER_TLS_CERTDIR: ""        # упрощённый dind без TLS (для внутреннего runner)
  script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" "$CI_REGISTRY"
    - docker build -t "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" .
    - docker tag "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" "$CI_REGISTRY_IMAGE:latest"
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"
    - docker push "$CI_REGISTRY_IMAGE:latest"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

> Переменные `CI_REGISTRY*` — встроенные. Свои секреты задавайте в **Settings > CI/CD > Variables** (с флагами Masked/Protected), а не в коде.

## 3. Кэширование зависимостей

```yaml
variables:
  npm_config_cache: "$CI_PROJECT_DIR/.npm"

cache:
  key:
    files:
      - package-lock.json
  paths:
    - .npm/

install:
  stage: build
  image: node:22
  script:
    - npm ci --cache .npm --prefer-offline
```

## 4. Условия запуска задач (rules)

```yaml
job:
  script: echo "ok"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"   # пайплайн из MR
      when: always
    - if: $CI_COMMIT_BRANCH == "main"                    # пуш в main
      when: on_success
    - if: $CI_COMMIT_TAG                                 # по тегу (релиз)
      when: always
    - when: never                                        # иначе не запускать
```

## 5. Деплой на сервер по SSH

Предварительно добавьте приватный ключ в CI/CD Variables (`SSH_PRIVATE_KEY`) и хост в `SSH_KNOWN_HOSTS`:

```yaml
deploy:
  stage: deploy
  image: alpine:3.20
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh && echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
  script:
    - ssh deploy@example.com "cd /opt/app && docker compose pull && docker compose up -d"
  environment:
    name: production
    url: https://app.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

## 6. Review Apps — временное окружение для каждой ветки

```yaml
review:
  stage: deploy
  script:
    - echo "Разворачиваю окружение для ветки $CI_COMMIT_REF_SLUG"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop-review
    auto_stop_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

stop-review:
  stage: deploy
  script:
    - echo "Удаляю окружение $CI_COMMIT_REF_SLUG"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
```

## 7. Полезные приёмы

```yaml
include:                        # подключение общих шаблонов
  - local: ci/build.yml

workflow:                       # не запускать пайплайн дважды (ветка + MR)
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS
      when: never
    - if: $CI_COMMIT_BRANCH

build:
  timeout: 30 minutes           # таймаут конкретной задачи
  retry: 2                      # повтор при падении инфраструктуры
  needs: ["install"]            # запуск без ожидания всего предыдущего stage
```

## 8. Полезные ссылки

* [Справочник ключевых слов .gitlab-ci.yml](https://docs.gitlab.com/ee/ci/yaml/)
* [Предопределённые переменные](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
* [Pipeline Editor](https://docs.gitlab.com/ee/ci/pipeline_editor/) — встроенный редактор с валидацией
