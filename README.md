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

## Unattended installation ISO

`autoinstall/user-data` defines a minimal Ubuntu Server 26.04 LTS installation:

* it refuses to start unless exactly one non-removable disk is present;
* it erases that disk and installs a direct GPT layout;
* it installs OpenSSH and permits the initial user to log in only with an SSH public key;
* it locks the local password after installation and grants that user passwordless `sudo` access;
* it powers the machine off after a successful installation.

The SSH public key is supplied at build time and is never stored in this repository. Build dependencies are `bash`, `openssl`, `sha256sum`, `xorriso`, and [mikefarah/yq](https://github.com/mikefarah/yq) v4; they are provided by the ISO builder container image.

The builder keeps the source ISO boot layout by loading it with `xorriso` and replaying its boot image configuration. It changes only `/boot/grub/grub.cfg` and the two `/autoinstall/` NoCloud files, so the source ISO's BIOS, UEFI, MBR, GPT, and El Torito settings are retained without assuming fixed internal boot file paths.

### ISO builder container image

`docker/iso-builder/Dockerfile` defines the shared build environment. Run `Publish ISO builder image` once before the first ISO build. It publishes `ghcr.io/imperatormarsa/ubuntumunkulus-iso-builder` with a mutable `v1` tag and an immutable `sha-<commit>` tag.

Make the package public in its GitHub Packages settings so `act` can pull it without credentials. The ISO workflow is pinned to an image digest; when publishing a new builder image, copy its digest from the publish workflow summary and update both the workflow and the local pull command in a separate commit.

Build the image through the `Build autoinstall ISO` GitHub Actions workflow. It is started manually and downloads the official ISO and its `SHA256SUMS` file itself. No build dependencies are installed during ISO creation.

Configure these repository secrets before running it:

| Secret | Value |
| --- | --- |
| `SOURCE_ISO_URL` | Direct URL of the official amd64 Ubuntu Server ISO. |
| `SOURCE_ISO_SHA256` | URL of the corresponding official `SHA256SUMS` file. The workflow finds the checksum for the ISO filename automatically. |
| `SSH_PUBLIC_KEY` | Public key that will be authorized for the initial user. |
| `HOSTNAME` | Hostname for the installed machine. |
| `USERNAME` | Name of the initial user. |

The generated ISO embeds the supplied public key. Treat it as machine-specific output; the workflow publishes it as `ubuntu-autoinstall-amd64`, and `build/` is intentionally not tracked. The original ISO remains unchanged.

### Local build with act

Install [nektos/act](https://github.com/nektos/act) and a Docker-compatible container runtime. Copy `act.secrets.example` to `act.secrets`, replace every value with the values for the target machine, then run:

```bash
docker pull ghcr.io/imperatormarsa/ubuntumunkulus-iso-builder@sha256:a4650404a6828aac0c12e954635c5b73dad8c5a880949489438906b1e3aff01a

act workflow_dispatch \
  --workflows .github/workflows/build-autoinstall-iso.yml \
  --secret-file act.secrets \
  --artifact-server-path "$PWD/build/artifacts"
```

`act.secrets` is ignored by Git. Docker caches the pulled builder image, so subsequent local ISO builds do not reinstall dependencies. The resulting artifact is written under `build/artifacts`; do not add it or the secrets file to the repository.

Inspect the resulting boot metadata when diagnosing boot failures:

```bash
xorriso -indev build/ubuntu-autoinstall-amd64.iso -report_el_torito plain
xorriso -indev build/ubuntu-autoinstall-amd64.iso -report_system_area plain
```

### QEMU smoke test

Download and extract the artifact to `build/ubuntu-autoinstall-amd64.iso`, then use an empty disposable virtual disk. The installer must stop before making changes if a second non-removable disk is attached.

```bash
qemu-img create -f qcow2 /tmp/ubuntu-autoinstall-test.qcow2 32G
qemu-system-x86_64 \
  -m 4G \
  -smp 2 \
  -drive file=/tmp/ubuntu-autoinstall-test.qcow2,format=qcow2 \
  -cdrom build/ubuntu-autoinstall-amd64.iso \
  -boot d
```

After the VM powers off, remove `-cdrom` and boot the same virtual disk. Verify SSH key login and `sudo -n true`; password authentication must be rejected.

Also boot the ISO in UEFI mode. On Ubuntu, install the `ovmf` package and adjust `OVMF_CODE` if the firmware is installed elsewhere:

```bash
OVMF_CODE=/usr/share/OVMF/OVMF_CODE.fd
test -r "$OVMF_CODE"

qemu-img create -f qcow2 /tmp/ubuntu-autoinstall-uefi-test.qcow2 32G
qemu-system-x86_64 \
  -m 4G \
  -smp 2 \
  -bios "$OVMF_CODE" \
  -drive file=/tmp/ubuntu-autoinstall-uefi-test.qcow2,format=qcow2 \
  -cdrom build/ubuntu-autoinstall-amd64.iso \
  -boot d
```

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
