---
description: Шпаргалка по Terraform
---

# Terraform

Terraform — это инструмент с открытым исходным кодом от компании HashiCorp, предназначенный для описания, создания и управления инфраструктурой в виде кода (Infrastructure as Code, IaC). Он позволяет определять инфраструктуру в файлах конфигурации и применять эти конфигурации для создания и изменения ресурсов в различных облачных и локальных средах.

### Установка Terraform

#### Скачивание бинарного файла

1. Перейдите на официальный сайт [Terraform](https://www.terraform.io/downloads).
2. Выберите версию для вашей операционной системы и скачайте архив.
3. Распакуйте архив и поместите бинарный файл `terraform` в директорию, которая находится в переменной окружения `PATH`.

#### Проверка установки

```bash
terraform version
```

### Основные понятия

* **Провайдеры (Providers)**: Отвечают за взаимодействие с API различных сервисов (AWS, Azure, Google Cloud, VMware и др.).
* **Ресурсы (Resources)**: Основные блоки инфраструктуры (виртуальные машины, сети, базы данных и др.).
* **Модули (Modules)**: Группы ресурсов, объединенные для переиспользования.
* **Переменные (Variables)**: Параметры, которые можно передавать в конфигурацию.
* **Выводы (Outputs)**: Позволяют экспортировать данные из конфигурации.

### Структура проекта Terraform

```plaintext
project/
├── main.tf          # Основной файл конфигурации
├── variables.tf     # Определение переменных
├── outputs.tf       # Определение выходных данных
├── terraform.tfvars # Значения переменных
└── modules/         # Директория с модулями (если используются)
```

### Файлы конфигурации

#### main.tf

Основной файл, где описываются провайдеры и ресурсы.

```hcl
# Определение провайдера
provider "aws" {
  region = "us-west-2"
}

# Определение ресурса
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

#### variables.tf

Определение переменных, которые можно использовать в конфигурации.

```hcl
variable "region" {
  description = "Регион AWS для развертывания ресурсов"
  type        = string
  default     = "us-west-2"
}
```

#### outputs.tf

Определение выходных данных, которые Terraform будет отображать после применения конфигурации.

```hcl
output "instance_ip" {
  description = "IP-адрес созданной EC2-инстанции"
  value       = aws_instance.example.public_ip
}
```

#### terraform.tfvars

Значения переменных, определенных в `variables.tf`.

```hcl
region = "us-east-1"
```

### Основные команды Terraform

*   **Инициализация проекта**

    ```bash
    terraform init
    ```

    Эта команда загружает необходимые провайдеры и инициализирует рабочую директорию.
*   **Проверка и формирование плана**

    ```bash
    terraform plan
    ```

    Создает и отображает план действий, которые Terraform выполнит для достижения желаемого состояния.
*   **Применение конфигурации**

    ```bash
    terraform apply
    ```

    Применяет конфигурацию и создает/изменяет/удаляет ресурсы в соответствии с планом.
*   **Удаление инфраструктуры**

    ```bash
    terraform destroy
    ```

    Удаляет все ресурсы, управляемые текущей конфигурацией Terraform.
*   **Форматирование кода**

    ```bash
    terraform fmt
    ```

    Форматирует файлы конфигурации в соответствии со стандартами стиля Terraform.
*   **Проверка синтаксиса**

    ```bash
    terraform validate
    ```

    Проверяет конфигурационные файлы на наличие синтаксических ошибок.

### Работа с переменными

#### Определение переменных в `variables.tf`

```hcl
variable "instance_type" {
  description = "Тип EC2-инстанции"
  type        = string
  default     = "t2.micro"
}
```

#### Использование переменных в конфигурации

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
}
```

#### Передача значений переменных

*   **Через файл `terraform.tfvars`**

    ```hcl
    instance_type = "t2.small"
    ```
*   **Через параметры командной строки**

    ```bash
    terraform apply -var="instance_type=t2.small"
    ```
*   **Через переменные окружения**

    ```bash
    export TF_VAR_instance_type="t2.small"
    ```

### Работа с состоянием (State)

Terraform хранит информацию о созданных ресурсах в файле состояния `terraform.tfstate`.

#### Особенности состояния

* **Локальное состояние**: По умолчанию состояние хранится в локальном файле.
* **Удаленное состояние**: Рекомендуется для командной работы. Состояние может храниться в облачных хранилищах (S3, GCS) или системах управления (Terraform Cloud, Consul).

#### Конфигурация удаленного состояния

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "path/to/my/key"
    region = "us-west-2"
  }
}
```

#### Блокировка состояния

При использовании удаленного состояния рекомендуется включить блокировку, чтобы избежать конфликтов при одновременном доступе.

### Модули

Модули позволяют переиспользовать код и структурировать конфигурацию.

#### Создание модуля

* Создайте директорию для модуля, например, `modules/ec2-instance`.
* Внутри создайте файлы `main.tf`, `variables.tf`, `outputs.tf`.

#### Использование модуля

```hcl
module "my_instance" {
  source = "./modules/ec2-instance"

  instance_type = "t2.medium"
  ami_id        = "ami-0c55b159cbfafe1f0"
}
```

#### Использование модулей из внешних источников

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "2.77.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b"]
}
```

### Провайдеры

Провайдеры отвечают за взаимодействие с различными сервисами. Каждый провайдер требует конфигурации.

#### Пример конфигурации провайдера AWS

```hcl
provider "aws" {
  region                  = var.region
  access_key              = var.aws_access_key
  secret_key              = var.aws_secret_key
  shared_credentials_file = "~/.aws/credentials"
  profile                 = "default"
}
```

#### Версионирование провайдеров

```hcl
provider "aws" {
  version = "~> 3.0"
  region  = var.region
}
```

### Управление зависимостями

Terraform автоматически определяет зависимости между ресурсами. Иногда требуется явно указать зависимости.

#### Использование `depends_on`

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  depends_on = [aws_security_group.sg]
}
```

### Функции и выражения

Terraform поддерживает различные функции для обработки данных.

#### Примеры функций

* **concat()**: Объединение списков
* **join()**: Объединение строк с разделителем
* **file()**: Чтение содержимого файла
* **lookup()**: Получение значения из карты (map)

#### Пример использования

```hcl
resource "aws_instance" "example" {
  ami           = lookup(var.ami_map, var.region)
  instance_type = var.instance_type
}
```

### Работа с конфигурационными файлами

#### Форматирование

```bash
terraform fmt
```

#### Синтаксическая проверка

```bash
terraform validate
```

#### Разбивка конфигурации на несколько файлов

Terraform автоматически объединяет все файлы с расширением `.tf` в одной директории.

### Terraform Workspace

Workspaces позволяют управлять несколькими независимыми экземплярами одной и той же конфигурации (например, для окружений dev, staging, prod).

#### Создание и переключение между workspace

```bash
terraform workspace new dev
terraform workspace select dev
```

#### Использование workspace в конфигурации

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket-${terraform.workspace}"
  acl    = "private"
}
```

### Лучшие практики

* **Храните состояние удаленно**: Используйте удаленные бекенды с блокировкой.
* **Используйте модули**: Для переиспользования и структурирования кода.
* **Версионируйте провайдеры и модули**: Явно указывайте версии для воспроизводимости.
* **Разделяйте окружения**: Используйте разные workspace или отдельные конфигурации.
* **Храните секреты безопасно**: Не храните пароли и ключи в репозиториях.
* **Автоматизируйте проверки**: Включите форматирование и валидацию в CI/CD процессы.

### Отладка и логирование

*   **Увеличение уровня логирования**

    ```bash
    export TF_LOG=TRACE
    terraform apply
    ```
*   **Логирование в файл**

    ```bash
    export TF_LOG=DEBUG
    export TF_LOG_PATH=./terraform.log
    terraform apply
    ```

### Интеграция с другими инструментами

* **Terraform Cloud/Enterprise**: Платформа для совместной работы с возможностями удаленного выполнения и управления состоянием.
* **Terragrunt**: Надстройка над Terraform для управления конфигурациями, разделения окружений и модулей.
* **Интеграция с CI/CD**: Используйте Terraform в конвейерах Jenkins, GitLab CI/CD, GitHub Actions и др.

### Полезные команды

*   **Получение выходных данных**

    ```bash
    terraform output
    terraform output instance_ip
    ```
*   **Обновление провайдеров и модулей**

    ```bash
    terraform init -upgrade
    ```
*   **Показ ресурсов состояния**

    ```bash
    terraform state list
    terraform state show aws_instance.example
    ```
*   **Удаление ресурса из состояния**

    ```bash
    terraform state rm aws_instance.example
    ```
*   **Перемещение ресурса в состояние**

    ```bash
    terraform state mv [options] SOURCE DESTINATION
    ```

### Ресурсы для дальнейшего изучения

* [Официальная документация Terraform](https://www.terraform.io/docs)
* [Руководство по языку конфигурации](https://www.terraform.io/docs/language/index.html)
* [Terraform Registry](https://registry.terraform.io/) — провайдеры и модули от сообщества
* [Практики HashiCorp по Terraform](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
* [HashiCorp Learn](https://learn.hashicorp.com/terraform) — обучающие материалы и туториалы

***

Эта шпаргалка охватывает ключевые аспекты работы с Terraform. Для углубленного изучения рекомендуется ознакомиться с официальной документацией и практиковаться в создании собственных конфигураций и модулей.
