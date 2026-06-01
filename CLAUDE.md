# fedora-init

Ansible playbook to set up a Fedora workstation. Run with `ansible-playbook fedora_init.yaml --ask-become-pass`.

## Structure

- `fedora_init.yaml` — main playbook: RPM Fusion repos, dnf-plugins-core, fact gathering, handler definitions. All other tasks delegated via `ansible.builtin.import_tasks`.
- `tasks/codecs.yaml` — ffmpeg swap and multimedia codec group.
- `tasks/nvidia.yaml` — NVIDIA drivers, services, nvtop. All tasks gated with `when: "'NVIDIA' in lspci_output.stdout"`.
- `tasks/packages.yaml` — dnf installs/removals, flatpaks, gnome-extensions-cli.
- `tasks/vscode.yaml` — VS Code repo and install.
- `tasks/brave.yaml` — Brave Origin browser repo and install.
- `tasks/gnome-extensions.yaml` — GNOME shell extensions via `gext`.
- `tasks/development.yaml` — build deps, pyenv, uv, Docker + NVIDIA container toolkit.

## Conventions

- `dnf-plugins-core` is installed once in `fedora_init.yaml` — do not add it to task files.
- RPM Fusion is set up in `fedora_init.yaml` — packages that need it (VLC, NVIDIA drivers, ffmpeg) can be added to task files without re-adding the repo.
- New user-facing apps go in `tasks/packages.yaml`. New developer tools go in `tasks/development.yaml`. Repo + package for a single app get their own task file (see vscode.yaml, brave.yaml).
- NVIDIA-conditional tasks must use `when: "'NVIDIA' in lspci_output.stdout"` — `lspci_output` is registered in the main playbook.
- Keep repo setup co-located with the package it serves (e.g. Docker repo lives in `development.yaml` next to the Docker install task).
- The current user is identified via `ansible_facts['user_id']` and their home via `ansible_facts['user_dir']` — no hardcoded username in the playbook.
- All modules must use FQCN (e.g. `ansible.builtin.dnf`, `ansible.builtin.systemd`). Never use short module names.
- Use `true`/`false` for all booleans. Never use `yes`/`no`.
- Service restarts triggered by config changes use handlers defined in `fedora_init.yaml`, not conditional tasks.
