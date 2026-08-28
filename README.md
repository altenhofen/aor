# altenhofen's own repository

Personal Arch Linux repository containing packages for my personal use.

Maintained by Augusto Altenhofen.

## Enable the repository

Add this repository after the official repositories in `/etc/pacman.conf`:

```ini
[altenhofen]
SigLevel = Optional TrustAll
Server = https://altenhofen.github.io/aor/$arch
```

Then refresh package databases:

```bash
sudo pacman -Sy
```

## Packages

- [`monochrome-bin`](monochrome-bin/): Desktop client for Monochrome music streaming.
- [`omarchy-windows-xp`](omarchy-windows-xp/): Windows XP Docker VM launcher for Omarchy.
