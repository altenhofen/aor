# Omarchy Windows XP

> This package has moved to [altenhofen/aor](https://github.com/altenhofen/aor).
> Use the `omarchy-windows-xp` package from that repository.

A small Arch Linux package that adds a Windows XP shortcut and command-line launcher to Omarchy. The VM runs through [dockur/windows](https://github.com/dockur/windows) in Docker and uses `xfreerdp3` for the desktop session, just like Omarchy's Windows 11 machine.

## Requirements

- Omarchy or another Arch Linux desktop session
- Docker Engine with access to `/dev/kvm` and `/dev/net/tun`
- `docker-compose`, `freerdp`, `libnotify`, and `uwsm`
- At least 2 GB available RAM and 32 GB free disk space
- A valid Windows XP license

Hardware virtualization must be enabled. The package does not contain or redistribute Windows XP; Dockur downloads the installation image when the VM is first started.

## Install

Build and install from a checkout:

```bash
makepkg -si
```

After installation, log out and back in if your account was newly added to the `docker` group.

The package installs:

- `/usr/bin/omarchy-windows-xp`
- `/usr/share/applications/windows-xp.desktop`

- `/usr/share/omarchy-windows-xp/compose.yaml`

To expose the XP command as `omarchy windows xp`, add this user-shell wrapper. It delegates every other Omarchy command to the stock dispatcher:

```bash
omarchy() {
  if [[ ${1:-} == windows && ${2:-} == xp ]]; then
    shift 2
    command omarchy-windows-xp "$@"
  else
    command /usr/share/omarchy/bin/omarchy "$@"
  fi
}
```

Put it in `~/.bashrc` (or the startup file for your shell), then open a new terminal. The package intentionally does not modify `/usr/share/omarchy`; that directory is owned by the Omarchy package and updates can replace its contents.

## Launch

Use either the application launcher:

```text
Super + Space → Windows XP
```

or the command line:

```bash
omarchy-windows-xp launch
```

The first launch downloads XP and performs the automatic installation. Use the web console during installation:

```text
http://127.0.0.1:8007/
```

After XP has finished booting, the launcher connects with `xfreerdp3` using fullscreen-style Omarchy behavior, dynamic resolution, clipboard, audio, microphone, and keyboard grabbing.

Default RDP credentials:

```text
User:     Docker
Password: admin
Address: 127.0.0.1:3390
```

The default storage directory is `~/WindowsXP`, matching Omarchy's Windows VM convention. Override it for another VM location:

```bash
OMARCHY_WINDOWS_XP_DIR="$HOME/other/xp" omarchy-windows-xp launch
```

## Commands

```bash
omarchy-windows-xp launch   # Start the VM and connect with RDP
omarchy-windows-xp stop     # Stop the VM
omarchy-windows-xp restart  # Restart the VM
omarchy-windows-xp status   # Show container status
omarchy-windows-xp logs     # Show Docker logs
omarchy-windows-xp config   # Print the Compose path
```

Closing the RDP session stops the container, matching the normal Omarchy Windows VM workflow.

## Resource and port overrides

The Compose template defaults to 2 GB RAM, 2 CPU cores, a 64 GB disk, web port 8007, and RDP port 3390. Override values for a launch with environment variables:

```bash
RAM_SIZE=4G CPU_CORES=4 DISK_SIZE=128G \
WEB_PORT=8010 RDP_PORT=3391 \
omarchy-windows-xp launch
```

Keep the RDP and web ports bound to localhost. Windows XP is obsolete and should not be exposed directly to a network or the Internet.

## Build a package

```bash
makepkg --nodeps --force
```

The generated package can be installed with:

```bash
sudo pacman -U omarchy-windows-xp-*.pkg.tar.zst
```

`.SRCINFO` is included for AUR-style distribution.
