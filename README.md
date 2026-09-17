# ubuntumunkulus

Набор Ansible-конфигураций и bootstrap-скриптов для воспроизводимой установки и настройки всех моих Ubuntu-машин.

Проект отвечает прежде всего за системный уровень и связывает между собой отдельные репозитории пользовательского окружения и development environments.

## Машины

Автоматизация используется для:

* личного ноутбука;
* рабочего ПК в офисе;
* mini-PC / homelab;
* VPS с VPN.

Общая конфигурация переиспользуется между машинами, а различия задаются через роли, группы и host-specific variables.

## Архитектура

Основным инструментом автоматизации системного уровня является **Ansible**.

Ответственность распределяется следующим образом:

```text
Ansible   → system configuration and orchestration
chezmoi   → user environment and dotfiles
Flatpak   → GUI applications
Homebrew  → CLI / TUI applications
Distrobox → isolated development environments
```

`chezmoi` и конфигурации Distrobox существуют как отдельные репозитории.

Этот репозиторий не дублирует их содержимое, а отвечает за подготовку системы и их подключение.

## System configuration — Ansible

Ansible приводит чистую Ubuntu-систему к необходимому базовому состоянию.

Через него выполняются:

* настройка APT;
* установка системных пакетов;
* настройка пользователей и групп;
* настройка SSH;
* настройка systemd;
* установка Flatpak;
* установка Homebrew;
* установка Distrobox и container runtime;
* настройка desktop/server-компонентов;
* machine-specific configuration;
* bootstrap внешнего chezmoi-репозитория;
* подготовка системы для использования отдельного Distrobox-репозитория.

Таким образом, Ansible является основной точкой входа для конфигурации новой машины.

## User environment — chezmoi

Dotfiles и пользовательская конфигурация находятся в отдельном репозитории и управляются через **chezmoi**.

Он отвечает за:

* shell configuration;
* Git configuration;
* terminal configuration;
* editors;
* CLI/TUI application configuration;
* scripts;
* пользовательские dotfiles.

Ansible устанавливает необходимые зависимости и инициирует bootstrap chezmoi, но непосредственно пользовательская конфигурация остаётся независимой от этого репозитория.

## Applications

Для пользовательского ПО по возможности используются дистрибутив-независимые способы установки.

```text
APT       → system packages and OS dependencies
Flatpak   → GUI applications
Homebrew  → CLI / TUI applications
```

### Flatpak

Большинство GUI-приложений устанавливается через Flatpak.

Это уменьшает зависимость от конкретной версии Ubuntu и количество сторонних APT-репозиториев.

### Homebrew

Большинство пользовательских CLI/TUI-инструментов устанавливается через Homebrew for Linux.

APT остаётся преимущественно для системных компонентов и зависимостей ОС.

## Development environments — Distrobox

Development environments не устанавливаются непосредственно в host-систему.

Для них используется **Distrobox**.

Конфигурация этих окружений находится в отдельном репозитории.

Примеры окружений:

```text
Python
PHP
Node.js
Rust
...
```

Каждое окружение может содержать собственные:

* runtime;
* compiler;
* SDK;
* package manager;
* development libraries;
* дополнительные development tools.

Основная Ubuntu-система при этом остаётся максимально чистой.

## Общая схема

```text
                           Ubuntu
                              │
                              ▼
                           Ansible
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
           APT             Flatpak          Homebrew
            │                 │                 │
      OS / system         GUI apps          CLI / TUI
      dependencies
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
         chezmoi repository          Distrobox repository
                │                           │
                ▼                           ▼
        user environment          development environments
```

## Репозитории

Архитектура намеренно разделена на несколько независимых репозиториев:

```text
ubuntu-automation
└── system provisioning / Ansible / orchestration

dotfiles
└── chezmoi-managed user configuration

distrobox
└── development environment definitions
```

Это позволяет развивать пользовательское окружение и development environments независимо от системной автоматизации.

## Пример структуры этого репозитория

```text
.
├── ansible/
│   ├── inventories/
│   │   ├── personal/
│   │   ├── work/
│   │   ├── homelab/
│   │   └── vps/
│   │
│   ├── group_vars/
│   ├── host_vars/
│   │
│   ├── roles/
│   │   ├── base/
│   │   ├── desktop/
│   │   ├── server/
│   │   ├── flatpak/
│   │   ├── homebrew/
│   │   ├── distrobox/
│   │   ├── chezmoi/
│   │   └── ...
│   │
│   └── playbooks/
│       ├── bootstrap.yml
│       ├── workstation.yml
│       ├── homelab.yml
│       └── vps.yml
│
├── scripts/
│   └── bootstrap.sh
│
├── ansible.cfg
└── README.md
```

Важно: роли `chezmoi` и `distrobox` здесь не содержат сами dotfiles или определения development environments. Они только устанавливают необходимые инструменты и подключают соответствующие внешние репозитории.

## Bootstrap

Цель — свести первоначальную настройку новой машины к минимальному количеству ручных действий.

Концептуально:

```text
Fresh Ubuntu
     │
     ▼
bootstrap.sh
     │
     ▼
Ansible
     │
     ├── system configuration
     ├── system packages
     ├── Flatpak
     ├── Homebrew
     ├── Distrobox
     │
     ├── bootstrap chezmoi repository
     │
     └── prepare Distrobox repository
```

После этого дальнейшее состояние машины должно определяться кодом, хранящимся в соответствующих репозиториях.

## Принципы

Проект строится вокруг нескольких принципов:

1. **Reproducibility** — новую машину можно воспроизвести из конфигурации.
2. **Idempotency** — повторный запуск Ansible безопасен.
3. **Separation of concerns** — system, user и development configuration разделены.
4. **Distribution independence** — пользовательское ПО по возможности не зависит от Ubuntu package repositories.
5. **Minimal host pollution** — development dependencies находятся в Distrobox.
6. **Single entry point** — Ansible выступает главным orchestrator для bootstrap новой системы.

## Цель проекта

Машина должна быть максимально заменяемой.

Переустановка Ubuntu, замена ноутбука или создание нового VPS не должны требовать ручного восстановления окружения.

Концептуально:

```text
machine =
    Ubuntu
  + system automation repository
  + chezmoi repository
  + distrobox repository
  + secrets
```

Физическая машина — не источник истины. Источником истины являются декларативные конфигурации в репозиториях.
