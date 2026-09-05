# WOOWTECH HA Add-on Mirror

This repository is a **defensive mirror** of an upstream Home Assistant add-on
store, hosted under `WOOWTECH` so our commercial HAOS fleet does not depend
on a single upstream repo or upstream container registry staying online.

**Do not develop against this repo.** All upstream changes flow in daily via
GitHub Actions. Any manual commits to `main` will be overwritten on the next
sync.

## What this mirror provides

| Branch | Purpose |
|--------|---------|
| `main` (default) | Upstream content + `.mirror/` overlay + `image:` fields rewritten to `ghcr.io/woowtech/ha-mirror-*`. This is the branch HAOS reads when you add this repo as an add-on store. |
| `upstream` | Pure `git push --mirror` copy of the upstream default branch. No modifications. Used for audit / recovery. |
| tags | All upstream tags preserved (force-updated nightly). |

## How to use in HAOS

```
Settings → Add-ons → Add-on Store → ⋮ → Repositories
Add: https://github.com/WOOWTECH/<this-repo>
```

Add-ons will show up with a **new slug prefix** because HAOS computes the
prefix from the repository URL. Existing installs from the upstream repo will
not migrate automatically — see `.mirror/README.md` for migration steps.

## Container images

All add-ons in this mirror have their `image:` field rewritten to point at
`ghcr.io/woowtech/ha-mirror-*`. The image-mirror workflow copies each upstream
image (all supported architectures, current version + `:latest`) into our GHCR
namespace, so add-on installs do not depend on the upstream registry either.

Upstream image → mirror image mapping is stored in `.mirror/image-map.json`
and refreshed on every sync.

## Automation

| Workflow | Schedule | Purpose |
|----------|----------|---------|
| `mirror-sync`  | `17 3 * * *` UTC | Pull upstream code, refresh `upstream` branch, patch configs, rebuild `main` |
| `image-mirror` | `47 4 * * *` UTC | Copy upstream container images to `ghcr.io/woowtech/ha-mirror-*` |

Manual runs: **Actions → mirror-sync / image-mirror → Run workflow**.

## Emergency: upstream is gone

If the upstream repo is deleted, the mirror keeps working indefinitely:

- Code: served from `main` (last synced snapshot)
- Images: served from `ghcr.io/woowtech/ha-mirror-*` (last synced tags)

Nothing on our HAOS fleet is affected.

## Files added by this mirror

| Path | Purpose |
|------|---------|
| `README.md` (this file) | Explains what the mirror is. |
| `.mirror/config.env` | Upstream URL, branch, and image prefix — sourced by workflows. |
| `.mirror/patch-image.py` | Rewrites `image:` fields; emits `.mirror/image-map.json`. |
| `.mirror/image-map.json` | Mapping consumed by `image-mirror` workflow. |
| `.mirror/last-sync.yaml` | Timestamp + upstream commit of last sync. |
| `.mirror/README.md` | Migration + operations notes for the mirror itself. |
| `.github/workflows/mirror-sync.yml` | Git sync automation. |
| `.github/workflows/image-mirror.yml` | Container image sync automation. |

Everything else in this repo comes verbatim from upstream.
