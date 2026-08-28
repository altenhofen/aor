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


## Last.fm popup issue

The Last.fm OAuth flow is implemented upstream in `js/settings.js`. It currently calls `window.open('', '_blank')` from the Tauri/WebKitGTK webview. When that returns `null`, the application shows `Popup blocked!` and returns without showing the credential fallback.

This is an application-level issue, not an Arch dependency or desktop-entry issue. The preferred fix is upstream: use Tauri's opener plugin or a native Tauri command to open the generated Last.fm URL in the system browser. A package recipe may carry a temporary source patch, but such patches must be reviewed against every upstream update.

## GitHub Pages repository build

`.github/workflows/repository.yml` builds and publishes the binary Arch repository. It triggers on pushes to `master` when any of these paths change:

- `.github/workflows/repository.yml`
- `monochrome-bin/**`
- `omarchy-windows-xp/**`

The workflow builds `monochrome-bin` and `omarchy-windows-xp`, regenerates the repository database, and publishes `site/` to the `gh-pages` branch.

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
