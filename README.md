# fedora-init

Ansible playbook to automate setting up a Fedora workstation from scratch.

## Layout

```
fedora-init/
├── ansible.cfg
├── hosts.yaml
├── fedora_init.yaml     # main playbook — system-level setup and entry point
└── tasks/
    ├── codecs.yaml          # ffmpeg swap and multimedia codec group
    ├── nvidia.yaml          # NVIDIA drivers, services, and nvtop
    ├── packages.yaml        # dnf packages, flatpaks (minimal + extras)
    ├── vscode.yaml          # Visual Studio Code
    ├── brave.yaml           # Brave Origin browser
    ├── fonts.yaml           # Microsoft Core Fonts
    ├── gnome-extensions.yaml # GNOME shell extensions via gext
    └── development.yaml     # build deps, pyenv, uv, Docker
```

### fedora_init.yaml

The main playbook handles foundational system setup that everything else depends on:

- Installs `dnf-plugins-core` (required by both Brave and Docker repo setup)
- Detects the Fedora version and whether an NVIDIA GPU is present
- Enables RPM Fusion (free + nonfree) repositories
- Delegates all other setup to task files via `ansible.builtin.import_tasks`
- Defines the `Restart Docker` handler used by the NVIDIA container toolkit configuration

### tasks/codecs.yaml

- Swaps `ffmpeg-free` for `ffmpeg` (skipped if already swapped)
- Updates the multimedia codec group

### tasks/nvidia.yaml

All NVIDIA-related setup, skipped automatically on non-NVIDIA systems:

- Installs NVIDIA drivers (`akmod-nvidia`, CUDA, power management, VA-API codec)
- Installs `nvtop` for GPU monitoring
- Enables `nvidia-suspend`, `nvidia-resume`, and `nvidia-hibernate` systemd services

### tasks/packages.yaml

General application installs and removals:

- Installs `gnome-tweaks`, `ffmpegthumbnailer`, `vlc`, `libreoffice-writer`, `libreoffice-calc`
- Removes unwanted GNOME apps (tour, contacts, maps, weather, help, and others)
- Adds Flathub and installs a minimal set of flatpak apps: Spotify, Flatseal, GNOME Extensions
- Installs extra flatpak apps tagged `extras` (skippable for a leaner setup): GIMP, Tiny Wii Backup Manager, PCSX2, Czkawka

### tasks/vscode.yaml

- Adds the Microsoft GPG key and yum repository
- Installs Visual Studio Code

### tasks/brave.yaml

- Adds the Brave Beta yum repository
- Installs Brave Origin browser

### tasks/fonts.yaml

- Installs font tooling dependencies (`curl`, `cabextract`, `xorg-x11-font-utils`, `mkfontscale`, `fontconfig`, `cpio`, `unzip`)
- Downloads the `msttcore-fonts-installer` RPM from SourceForge and verifies its SHA256 checksum
- Installs the RPM with `--nodigest --nofiledigest` (required because this 2013 community package lacks the digest metadata DNF5 expects); skipped if already installed

### tasks/gnome-extensions.yaml

Installs GNOME shell extensions via `gext install`:

- `dash-to-dock@micxgx.gmail.com`
- `randomwallpaper@iflow.space`
- `blur-my-shell@aunetx`
- `accent-directories@taiwbi.com`

> **Note:** `gext install` also attempts to enable extensions, which requires a running GNOME session. If run before first login, install succeeds but enabling may fail silently. Re-run or enable manually with `gext enable <uuid>` after logging in.

### tasks/development.yaml

Developer toolchain setup:

- Installs C/Python build dependencies (`gcc`, `make`, `*-devel` packages, etc.)
- Installs [pyenv](https://github.com/pyenv/pyenv) (skipped if already installed), runs `pyenv update`, and configures it in `.bashrc` and `.bash_profile`
- Installs [uv](https://github.com/astral-sh/uv) and configures shell completion
- Installs Docker CE and (if NVIDIA detected) the NVIDIA Container Toolkit
- Configures the NVIDIA runtime for Docker and triggers a Docker restart via handler
- Adds the current user to the `docker` group

## Tags

Two tags allow trimming the install for a leaner setup:

- `extras` — optional flatpak apps in `tasks/packages.yaml` (GIMP, Tiny Wii Backup Manager, PCSX2, Czkawka)
- `development` — everything in `tasks/development.yaml` and `tasks/vscode.yaml` (build deps, pyenv, uv, Docker, VS Code)

Skip either or both with `--skip-tags`:

```
ansible-playbook fedora_init.yaml --skip-tags extras --ask-become-pass
ansible-playbook fedora_init.yaml --skip-tags extras,development --ask-become-pass
```

## Prerequisites

```
sudo dnf -y update
```

```
sudo dnf install ansible -y
ansible-galaxy collection install community.general
```

Then reboot before running the playbook so the updated kernel is active.

## Run

> [!CAUTION] DO NOT RUN AS ROOT

```
ansible-playbook fedora_init.yaml --ask-become-pass
```

Reboot after the playbook completes (required for NVIDIA drivers and group membership changes to take effect).
