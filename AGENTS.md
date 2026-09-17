# Repository Guidance

## Current State

- This repository currently contains only `README.md`; the Ansible layout and `bootstrap.sh` shown there are a target example, not files or runnable commands in this checkout.
- Do not invent inventory names, playbook paths, bootstrap commands, or verification commands until their configuration is added to the repository.

## Intended Boundaries

- This repository owns Ubuntu system provisioning and orchestration; Ansible is the intended primary entry point for machine bootstrap.
- Keep user dotfiles and configuration in the separate chezmoi repository. This repository should only install or bootstrap chezmoi.
- Keep development environment definitions in the separate Distrobox repository. Do not place language runtimes, SDKs, or development dependencies directly on the host unless they are required system dependencies.
- Follow the documented package split: APT for OS/system dependencies, Flatpak for GUI applications, and Homebrew for CLI/TUI applications.
- Provisioning should remain reproducible and idempotent across personal, work, homelab, and VPS machines; express machine differences through roles, groups, and host-specific variables once the Ansible structure exists.
