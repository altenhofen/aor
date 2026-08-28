# AGENTS.md

## Repository purpose

This repository contains Arch Linux packages published through the `altenhofen` repository on GitHub Pages.

Package recipes live in one directory per package. Each package directory contains a `PKGBUILD`; tracked source packages also contain a generated `.SRCINFO`.

## Creating a package

1. Create a directory named exactly after `pkgname`.
2. Add a valid `PKGBUILD`.
3. Generate metadata from that directory:

   ```bash
   makepkg --printsrcinfo > .SRCINFO
   ```

4. Validate the recipe:

   ```bash
   bash -n PKGBUILD
   makepkg --printsrcinfo >/tmp/package.SRCINFO
   cmp -s /tmp/package.SRCINFO .SRCINFO
   makepkg --nobuild --nodeps
   ```

5. Remove temporary `src/`, `pkg/`, and downloaded source directories before committing unless they are intentionally tracked.
6. Commit both `PKGBUILD` and `.SRCINFO`.
7. Push to `master` to trigger the repository build workflow.

Use existing package recipes as the convention. Keep runtime dependencies in `depends` and build-only tools in `makedepends`. Use `provides` and `conflicts` when a package replaces another package.

## Monochrome packages

### `monochrome-bin`

`monochrome-bin` is a binary package. It downloads the latest upstream Debian release metadata, retrieves the matching `.deb`, and extracts its `data.tar.*` member into `$pkgdir`.

Do not add application behavior to this recipe. The application is embedded in the upstream Tauri binary.

### `monochrome-git`

`monochrome-git` builds the upstream `main` branch:

- Source: `https://github.com/monochrome-music/desktop-app.git`
- Build tools: `bun`, `rust`, and `dpkg`
- Runtime dependencies: `libayatana-appindicator`, `webkit2gtk-4.1`, and `gtk3`
- Build sequence: `bun install --frozen-lockfile`, then `bun run tauri build --bundles deb`
- Package output: the generated Tauri Debian bundle under `src-tauri/target/release/bundle/deb/`

The generated Debian archive must be unpacked by extracting its `data.tar.*` member; a Debian package is an `ar` archive and cannot be passed directly to `bsdtar` as the package payload.

The package uses a git-style version generated from the source checkout:

```bash
printf 'r%s.g%s' "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
```

`monochrome-git` conflicts with `monochrome-bin`. Do not install both simultaneously.

## Downstream source patches

When a downstream patch is necessary, apply it in `prepare()` after the git source is checked out and before `build()` runs. Keep the patch narrow and anchored to stable upstream source.

The current Monochrome window-decoration patch is:

```bash
prepare() {
    cd "$srcdir/desktop-app"
    sed -i '/\.resizable(true)$/a\            .decorations(false)' src-tauri/src/lib.rs
}
```

`.decorations(false)` removes the native titlebar and border. It may also remove the normal window drag area if the application does not provide a custom draggable region.

Do not patch the installed Tauri ELF after packaging. The frontend and Rust code are embedded in the built application, so behavior changes should be made in the source build or upstream.

## Last.fm popup issue

The Last.fm OAuth flow is implemented upstream in `js/settings.js`. It currently calls `window.open('', '_blank')` from the Tauri/WebKitGTK webview. When that returns `null`, the application shows `Popup blocked!` and returns without showing the credential fallback.

This is an application-level issue, not an Arch dependency or desktop-entry issue. The preferred fix is upstream: use Tauri's opener plugin or a native Tauri command to open the generated Last.fm URL in the system browser. A package recipe may carry a temporary source patch, but such patches must be reviewed against every upstream update.

## GitHub Pages repository build

`.github/workflows/repository.yml` builds and publishes the binary Arch repository. It triggers on pushes to `master` when any of these paths change:

- `.github/workflows/repository.yml`
- `monochrome-bin/**`
- `omarchy-windows-xp/**`

The workflow builds only the package directories listed in its `for package in ...` loop, regenerates the repository database, and publishes `site/` to the `gh-pages` branch.

`monochrome-git` is intentionally not built by GitHub Actions. It is an AUR-style source package: users clone this repository and build it locally:

```bash
git clone https://github.com/altenhofen/aor.git
cd aor/monochrome-git
makepkg -si
```

To update it:

```bash
git pull
makepkg -si
```

A normal push to `master` triggers the binary repository workflow. To run it manually:

```bash
gh workflow run repository.yml --ref master
```

Monitor runs at:

```text
https://github.com/altenhofen/aor/actions
```

A local commit does not trigger GitHub Actions until it is pushed.

## Commit requirements

Before committing:

- Ensure `PKGBUILD` passes `bash -n`.
- Regenerate and compare `.SRCINFO`.
- Run `makepkg --nobuild --nodeps` when source retrieval is practical.
- Run `git diff --check`.
- Confirm the workflow includes the new package directory.
- Commit the recipe and metadata together.
- Push `master` when the published repository should update.
