# fedora-init

Ansible playbook to automate setting up a Fedora workstation from scratch.

## Layout

```
fedora-init/
├── fedora_init.yaml     # main playbook — system-level setup and entry point
└── tasks/
    ├── nvidia.yaml      # NVIDIA drivers, services, and nvtop
    ├── packages.yaml    # user-facing apps: dnf packages, VS Code, Brave, flatpaks
    └── development.yaml # developer tools: build deps, pyenv, uv, Docker
```

### fedora_init.yaml

The main playbook handles foundational system setup that everything else depends on:

- Installs `dnf-plugins-core` (required by both Brave and Docker repo setup)
- Detects the Fedora version and whether an NVIDIA GPU is present
- Enables RPM Fusion (free + nonfree) repositories
- Swaps `ffmpeg-free` for `ffmpeg` and updates the multimedia codec group

It then delegates to the task files via `import_tasks`.

### tasks/nvidia.yaml

All NVIDIA-related setup, skipped automatically on non-NVIDIA systems:

- Installs NVIDIA drivers (`akmod-nvidia`, CUDA, power management, VA-API codec)
- Installs `nvtop` for GPU monitoring
- Enables `nvidia-suspend`, `nvidia-resume`, and `nvidia-hibernate` systemd services

### tasks/packages.yaml

General application installs and removals:

- Installs `gnome-tweaks`, `ffmpegthumbnailer`, `vlc`, `libreoffice-writer`, `libreoffice-calc`
- Removes unwanted GNOME apps (tour, contacts, maps, weather, help)
- Installs Visual Studio Code (via Microsoft repo)
- Installs Brave Origin browser (via Brave beta repo)
- Adds Flathub and installs flatpak apps:
  - Spotify, GIMP, mpv, Flatseal, GNOME Extensions
  - Tiny Wii Backup Manager, PCSX2, Czkawka

### tasks/development.yaml

Developer toolchain setup:

- Installs C/Python build dependencies (`gcc`, `make`, `*-devel` packages, etc.)
- Installs [pyenv](https://github.com/pyenv/pyenv) and configures it in `.bashrc` and `.bash_profile`
- Installs [uv](https://github.com/astral-sh/uv) and configures shell completion
- Installs Docker CE and (if NVIDIA detected) the NVIDIA Container Toolkit
- Configures the NVIDIA runtime for Docker (if NVIDIA detected)
- Adds the current user to the `docker` group

## Prerequisites

```
sudo dnf -y update
```

```
sudo dnf install ansible-playbook -y
sudo ansible-galaxy collection install community.general
```

Then reboot before running the playbook so the updated kernel is active.

## Run

```
ansible-playbook fedora_init.yaml --ask-become-pass
```

Reboot after the playbook completes (required for NVIDIA drivers and group membership changes to take effect).
