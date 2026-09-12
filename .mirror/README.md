# `.mirror/` — mirror operations

Everything here is WOOWTECH-added, not from upstream. Do not delete these
files or the automation will break.

## Files

- **`config.env`** — sourced by workflows. Change here if upstream branch
  ever renames (e.g. `master` → `main`).
- **`patch-image.py`** — rewrites `image:` fields in every add-on config so
  HAOS pulls from `ghcr.io/woowtech/ha-mirror-*` instead of upstream ghcr.
  Also emits `image-map.json`.
- **`image-map.json`** — machine-readable list of `(upstream image, arches,
  version, mirror image)` tuples. Consumed by `.github/workflows/image-mirror.yml`.
- **`last-sync.yaml`** — timestamp + upstream commit of the last successful
  mirror-sync run.

## Image naming

Sanitizer: `lowercase; '/' → '-'; '.' → '-'`

| Upstream image | Mirror image |
|----------------|--------------|
| `ghcr.io/hassio-addons/ssh` | `ghcr.io/woowtech/ha-mirror-ghcr-io-hassio-addons-ssh` |
| `ghcr.io/riddix/home-assistant-matter-hub-addon` | `ghcr.io/woowtech/ha-mirror-ghcr-io-riddix-home-assistant-matter-hub-addon` |
| `ghcr.io/music-assistant/server` | `ghcr.io/woowtech/ha-mirror-ghcr-io-music-assistant-server` |

HAOS Supervisor appends `-<arch>:<version>` when pulling, so the actual pull
URL becomes e.g. `ghcr.io/woowtech/ha-mirror-ghcr-io-hassio-addons-ssh-amd64:24.0.1`.

## Migrating an existing HAOS add-on to the mirror

Slug prefix changes when you swap store URLs (Supervisor hashes the URL). The
add-on will look like a brand-new install to Supervisor, so **installed data
is not automatically carried over**.

```bash
# On HAOS
# 1. Snapshot current add-on data
ha backups new --addons <old_slug>

# 2. Uninstall old add-on
ha apps uninstall <old_slug>

# 3. Remove old store repo, add WOOWTECH mirror
ha store remove <old_repo_url>
ha store add    https://github.com/WOOWTECH/<mirror-repo>

# 4. Install from mirror
ha apps install <new_slug>          # new slug uses WOOWTECH repo hash

# 5. Restore data (requires manual copy of /data — depends on add-on)
```

For new machines, just add the WOOWTECH mirror URL from day one and skip the
migration entirely.

## Verifying a sync

```bash
gh run list --workflow mirror-sync  --limit 3
gh run list --workflow image-mirror --limit 3
cat .mirror/last-sync.yaml
```

Both workflows can be run manually from the Actions tab (`Run workflow`).
