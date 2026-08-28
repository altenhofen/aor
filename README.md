# altenhofen's own repository

Personal Arch Linux repository containing packages maintained by Augusto Altenhofen.

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

Every push to `master` runs `.github/workflows/repository.yml`. GitHub Actions builds both packages in Arch Linux, regenerates the `altenhofen` repository database, and publishes `x86_64/` to the `gh-pages` branch. Enable GitHub Pages for the repository with `gh-pages` as its source.
