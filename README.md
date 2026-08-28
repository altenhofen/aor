# altenhofen's Arch Linux repository

Personal Arch Linux repository containing packages maintained by Augusto Altenhofen.

## Enable the repository

Add this repository after the official repositories in `/etc/pacman.conf`:

```ini
[altenhofen]
SigLevel = Optional TrustAll
Server = https://raw.githubusercontent.com/altenhofen/aor/master/$arch
```

Then refresh package databases:

```bash
sudo pacman -Sy
```

## Packages

- [`monochrome-git`](monochrome-git/): Desktop client for Monochrome music streaming.
- [`omarchy-windows-xp`](omarchy-windows-xp/): Windows XP Docker VM launcher for Omarchy.

Package source directories contain the corresponding `PKGBUILD` and `.SRCINFO` files. The repository database and built packages are published under `x86_64/`.
