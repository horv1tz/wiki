---
description: Удобная и красивая консоль Linux
---

# Терминал

### Установка ZSH

#### Для RHEL/Fedora

```bash
sudo dnf install zsh
```

#### Для Ubuntu/Debian

```bash
sudo apt install zsh
```

#### Для ALT Linux

```bash
epm install zsh
```

### Установка Oh-My-Zsh

Официальный сайт: [Oh-My-Zsh](https://ohmyz.sh/)

#### Установка зависимостей

Перед установкой Oh-My-Zsh необходимо установить необходимые зависимости. Выполните соответствующую команду для вашего дистрибутива:

*   **RHEL/Fedora**

    ```bash
    sudo dnf install git curl wget
    ```
*   **Ubuntu/Debian**

    ```bash
    sudo apt install git curl wget
    ```
*   **ALT Linux**

    ```bash
    epm install git curl wget
    ```

#### Установка с помощью Curl

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### Установка с помощью Wget

```bash
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

### Выбор темы

Oh-My-Zsh предоставляет множество тем для настройки внешнего вида вашего терминала. Подробная информация доступна на [GitHub Wiki - Themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes).

Пример выбора темы в файле `.zshrc`:

```bash
ZSH_THEME="agnoster"
```

### Выбор плагинов

Плагины расширяют функциональность ZSH, добавляя автодополнение, подсветку синтаксиса и другие полезные функции. Подробная информация доступна на [GitHub Wiki - Plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins).

#### Список рекомендуемых плагинов

```bash
plugins=(
    gitfast
    docker
    zsh-autosuggestions
    zsh-syntax-highlighting
    zsh-completions
    docker-compose
    z
    zsh-interactive-cd
    extract
    colored-man-pages
)
```

Для плагина плагина **zsh-interactive-cd** необходим пакет **fzf**

#### Установка дополнительных плагинов

Для некоторых плагинов требуется их клонирование из репозиториев. Добавьте следующий скрипт в ваш `.zshrc` или выполните его после установки Oh-My-Zsh:

```bash
# Установка zsh-autosuggestions
if [ ! -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions" ]; then
  git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
fi

# Установка zsh-syntax-highlighting
if [ ! -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting" ]; then
  git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
fi

# Установка zsh-completions
if [ ! -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-completions" ]; then
  git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-completions
fi
```

После внесения изменений в `.zshrc`, примените их, перезапустив терминал или выполнив команду:

```bash
source ~/.zshrc
```

***

Теперь ваш ZSH настроен с Oh-My-Zsh, выбранной темой и необходимыми плагинами для повышения продуктивности и удобства работы в терминале.
