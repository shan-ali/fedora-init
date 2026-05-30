# fedora-init

Ansible playbook to set up a Fedora workstation. Run with `sudo ansible-playbook fedora_init.yaml`.

## Structure

- `fedora_init.yaml` — main playbook, system-level setup only (repos, codecs, NVIDIA drivers). All other tasks are delegated via `import_tasks`.
- `tasks/nvidia.yaml` — NVIDIA drivers, services, nvtop. All tasks gated with `when: "'NVIDIA' in lspci_output.stdout"`.
- `tasks/packages.yaml` — user-facing apps: dnf installs/removals, VS Code, Brave, flatpaks.
- `tasks/development.yaml` — developer tools: build deps, pyenv, uv, Docker + NVIDIA container toolkit.

## Conventions

- `dnf-plugins-core` is installed once in `fedora_init.yaml` — do not add it to task files.
- RPM Fusion is set up in `fedora_init.yaml` — packages that need it (VLC, NVIDIA drivers, ffmpeg) can be added to task files without re-adding the repo.
- New user-facing apps go in `tasks/packages.yaml`. New developer tools go in `tasks/development.yaml`.
- NVIDIA-conditional tasks must use `when: "'NVIDIA' in lspci_output.stdout"` — `lspci_output` is registered in the main playbook.
- Keep repo setup co-located with the package it serves (e.g. Docker repo lives in `development.yaml` next to the Docker install task).
- The current user is defined as `current_user_name: "shan"` in the main playbook vars.
