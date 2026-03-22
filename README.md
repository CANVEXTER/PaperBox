>This is just a tinkering/ hobby project, please don't expect anything industrial grade. feel free to criticize, contribute and suggest. 

## Goal of this project

This project is basically a **systemd-nspawn configuration**, which means it works out of the box on most Linux distributions without needing Docker or Podman.

The goal is to create a **secondary, isolated userspace environment** inside a Linux system for development, testing, or installing software without affecting the host.

Modern distros—especially rolling or heavily customized ones like CachyOS or Manjaro—can break due to package changes or dependency conflicts. Tools like Python are tightly integrated into the system, and modifying them can cause issues. On top of that, different default shells (fish, zsh vs bash) can lead to inconsistencies since most development workflows assume bash.

Existing tools like Distrobox solve many of these problems which btw is really good and recommend to trying out, but they rely on container engines, which adds overhead and can limit system-level access or nested container use.

This project aims to provide a **lighter, more native alternative** with minimal abstraction, proper isolation, and the ability to run system-level software while keeping the host system clean and stable.
## Summary

This project aims to provide:

> a minimal, systemd-native development environment that is isolated, reproducible, and behaves like a normal Linux system without interfering with the host
---

# 1. install required tools (host)

```bash
sudo pacman -S arch-install-scripts
```

# 2. create container rootfs

```bash
sudo mkdir -p /var/lib/machines/dev
sudo pacstrap -c /var/lib/machines/dev base base-devel bash
```

# 3. enter container (basic test)

```bash
sudo systemd-nspawn -D /var/lib/machines/dev
```

inside:

```bash
pacman -Syu
```

exit after test

# 4. install basic + GUI test tools

enter again:

```bash
sudo systemd-nspawn -D /var/lib/machines/dev
```

inside:

```bash
pacman -S xorg-xeyes tk
```

exit

# 5. create launcher script

```bash
nano ~/.local/bin/dev
```

## script (working base version)

```bash
#!/bin/bash

MYUID=$(id -u)

xhost +SI:localuser:root >/dev/null 2>&1

sudo -E systemd-nspawn \
  -D /var/lib/machines/dev \
  --console=interactive \
  --background= \
  --bind=/run/user/$MYUID \
  --bind=/tmp/.X11-unix \
  --bind=/dev/dri \
  --setenv=XDG_RUNTIME_DIR=/run/user/$MYUID \
  --setenv=WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
  --setenv=DISPLAY=$DISPLAY \
  /bin/bash
```

make executable:

```bash
chmod +x ~/.local/bin/dev
```

# 6. run container

```bash
dev
```

# 7. test GUI

inside container:

```bash
xeyes
```

✅ window should open on host

# 8. fix display permission (one-time on host)

if you get auth error:

```bash
sudo pacman -S xorg-xhost
xhost +SI:localuser:root
```

---
# 9. known behavior

- running as **root**
- GUI works because:
	- display socket passed
	- runtime dir mapped
	- xhost allows access

---

# ⚠️ limitations (current stage)

- root user only - methods exists but still looking for a proper way since maapplications won't properly run
- no proper user separation yet
- environment tied to root
- not production-clean (but works)

> Note that all of this issues will be addressed as the progress continues.
> I'll also soon provide 
