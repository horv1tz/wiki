---
description: Шпаргалка по Ansible
---

# Ansible

Ansible — это инструмент с открытым исходным кодом для управления конфигурацией, развертывания приложений и автоматизации IT-задач. Он использует простой язык YAML, позволяющий описывать инфраструктуру как код (Infrastructure as Code, IaC).

### Установка Ansible

#### На Ubuntu/Debian

```bash
sudo apt update
sudo apt install ansible
```

#### На CentOS/RHEL

```bash
sudo yum install epel-release
sudo yum install ansible
```

#### Проверка версии

```bash
ansible --version
```

### Инвентарь (Inventory)

Инвентарь — это список узлов, которыми Ansible будет управлять.

#### Статический инвентарь

Файл `hosts` или `inventory`:

```
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com
```

#### Динамический инвентарь

Можно использовать скрипты или плагины для динамического получения списка узлов из облачных провайдеров (AWS, GCP, Azure и др.).

### Ad-Hoc команды

Используются для выполнения одноразовых задач на удаленных узлах.

#### Общий синтаксис

```bash
ansible <pattern> -m <module> -a "<arguments>"
```

#### Примеры

*   Проверка доступности узлов:

    ```bash
    ansible all -m ping
    ```
*   Выполнение команды на удаленных узлах:

    ```bash
    ansible webservers -m shell -a "uname -a"
    ```

### Playbooks

Playbooks — это сценарии, написанные на YAML, описывающие желаемое состояние системы.

#### Структура Playbook

```yaml
- hosts: target_hosts
  become: yes  # Повышение привилегий до root
  tasks:
    - name: Описание задачи
      module_name:
        option1: value1
        option2: value2
```

#### Пример простого Playbook

```yaml
- hosts: webservers
  become: yes
  tasks:
    - name: Установить Nginx
      apt:
        name: nginx
        state: present
```

#### Запуск Playbook

```bash
ansible-playbook playbook.yml
```

### Модули

Ansible имеет множество встроенных модулей для управления различными аспектами системы.

#### Некоторые распространенные модули

* **package/apt/yum/dnf**: Управление пакетами.
* **service/systemd**: Управление системными сервисами.
* **user**: Управление пользователями.
* **copy**: Копирование файлов на удаленные узлы.
* **file**: Управление правами доступа к файлам и директориям.
* **template**: Развертывание файлов-шаблонов с использованием Jinja2.

#### Пример использования модуля

```yaml
- name: Копировать файл конфигурации
  copy:
    src: ./nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: 0644
```

### Переменные

Переменные позволяют параметризовать Playbooks.

#### Определение переменных

*   Внутри Playbook:

    ```yaml
    vars:
      http_port: 80
      max_clients: 200
    ```
* В отдельных файлах (`group_vars`, `host_vars`).

#### Использование переменных

```yaml
- name: Настроить Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

В шаблоне `nginx.conf.j2`:

```nginx
server {
    listen {{ http_port }};
    # Другие настройки
}
```

### Факты (Facts)

Ansible собирает системную информацию об узлах, называемую "фактами".

#### Использование фактов

```yaml
- name: Показать версию ОС
  debug:
    msg: "Версия ОС: {{ ansible_distribution_version }}"
```

### Условия (Conditionals)

Позволяют выполнять задачи на основе определенных условий.

#### Пример использования

```yaml
- name: Установить Apache на Debian
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"
```

### Циклы (Loops)

Позволяют выполнять одну и ту же задачу с разными параметрами.

#### Пример использования

```yaml
- name: Создать пользователей
  user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie
```

### Регистры (Register)

Сохраняют вывод команды для последующего использования.

#### Пример использования

```yaml
- name: Проверить доступность сервиса
  shell: systemctl is-active nginx
  register: nginx_status
  ignore_errors: yes

- name: Вывести статус сервиса
  debug:
    msg: "Статус Nginx: {{ nginx_status.stdout }}"
```

### Роли (Roles)

Роли помогают структурировать Playbooks и позволяют переиспользовать код.

#### Структура роли

```
roles/
└── role_name/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    └── README.md
```

#### Использование роли в Playbook

```yaml
- hosts: webservers
  roles:
    - role_name
```

### Обработчики (Handlers)

Используются для выполнения действий при возникновении определенных событий, обычно уведомлений `notify`.

#### Пример использования

```yaml
- name: Изменить конфигурацию Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - Перезапустить Nginx

handlers:
  - name: Перезапустить Nginx
    service:
      name: nginx
      state: restarted
```

### Шаблоны (Templates)

Ansible использует Jinja2 для шаблонизации файлов.

#### Пример шаблона

`nginx.conf.j2`:

```nginx
server {
    listen {{ http_port }};
    server_name {{ domain_name }};
    # Другие настройки
}
```

#### Использование шаблона в Playbook

```yaml
- name: Развернуть конфигурацию Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

### Ansible Vault

Ansible Vault позволяет шифровать секретные данные, такие как пароли или ключи API.

#### Создание зашифрованного файла

```bash
ansible-vault create secrets.yml
```

#### Редактирование зашифрованного файла

```bash
ansible-vault edit secrets.yml
```

#### Использование зашифрованных переменных

```yaml
- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Использовать секретный пароль
      debug:
        msg: "Пароль: {{ secret_password }}"
```

#### Запуск Playbook с Ansible Vault

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

### Фильтры и Функции

Ansible поддерживает различные фильтры для преобразования данных.

#### Пример использования фильтров

```yaml
- name: Преобразовать строку в нижний регистр
  debug:
    msg: "{{ 'HELLO WORLD' | lower }}"
```

### Практики лучшего использования

* **Организуйте код**: Используйте роли и коллекции для структурирования.
* **Используйте версии**: Храните Playbooks в системах контроля версий (Git).
* **Документируйте**: Добавляйте описания к задачам и ролям.
* **Тестируйте**: Проверяйте Playbooks перед применением в продакшене.
* **Следите за обновлениями**: Регулярно обновляйте Ansible и модули.

### Полезные команды

*   **Проверка синтаксиса Playbook**

    ```bash
    ansible-playbook playbook.yml --syntax-check
    ```
*   **Просмотр фактов об узлах**

    ```bash
    ansible all -m setup
    ```
*   **Запуск Playbook в режиме проверки (Dry Run)**

    ```bash
    ansible-playbook playbook.yml --check
    ```
*   **Увеличение уровня подробности вывода**

    ```bash
    ansible-playbook playbook.yml -v      # Уровень 1
    ansible-playbook playbook.yml -vvv    # Уровень 3
    ansible-playbook playbook.yml -vvvv   # Уровень 4 (максимальный)
    ```
*   **Указание пользовательского инвентаря**

    ```bash
    ansible-playbook -i custom_inventory playbook.yml
    ```

### Ресурсы для дальнейшего изучения

* [Официальная документация Ansible](https://docs.ansible.com/)
* [Ansible Galaxy](https://galaxy.ansible.com/) — репозиторий ролей и коллекций
* [Jinja2 шаблоны](https://jinja.palletsprojects.com/) — документация по шаблонам
* [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

***

Эта шпаргалка покрывает основные аспекты работы с Ansible. Для углубленного понимания рекомендуется ознакомиться с официальной документацией и практиковаться в написании собственных Playbooks и ролей.
