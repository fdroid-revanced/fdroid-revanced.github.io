# fdroid-revanced.github.io

Unofficial [F-Droid](https://f-droid.org/) **simple binary** repository of [ReVanced](https://revanced.app/)-patched apps.

Built by [**revancedbot**](https://github.com/lucasew/revancedbot) on GitHub Actions and served via GitHub Pages.

## Add this repo in F-Droid

1. Open F-Droid → **Settings** → **Repositories** → **+**
2. URL:

```text
https://fdroid-revanced.github.io/repo
```

3. Enable the repository and update indexes.

> This is **unofficial**. You install third-party signed APKs at your own risk.

## How it works

| Piece | Role |
|--------|------|
| `revancedbot.yaml` | Authority config (name, URL, downloaders) |
| `repo/` | APKs + F-Droid indexes (published) |
| `metadata/` | Package descriptions (published) |
| `config.yml` | Generated each run — **not** committed |
| GitHub Actions | Weekly + manual rebuild |

Pipeline (on `ubuntu-latest`):

1. Install Java, `fdroid`, `aapt`, `apksigner`, and `revancedbot`
2. `revancedbot run .` with secret `REVANCEDBOT_SIGNING`
3. Commit `repo/` + `metadata/` to `main` (Pages)

Signing is one pasteable blob from:

```bash
revancedbot keys generate
# put the single stdout line into repo secret REVANCEDBOT_SIGNING
```

## Operator setup (once)

1. **Secret** `REVANCEDBOT_SIGNING` — from `revancedbot keys generate` (same key forever for app updates).
2. **GitHub Pages**: Settings → Pages → Deploy from branch **`main`** / root (or whatever this org uses for `*.github.io`).
3. **Actions** write permission: Settings → Actions → General → Workflow permissions → **Read and write**.
4. Run workflow **Build F-Droid repo** manually first (`workflow_dispatch`). Optional **smoke** input builds one package only.

## Local rebuild

```bash
mise install
mise run install-revancedbot
export REVANCEDBOT_SIGNING='…'   # same secret
mise run build-repo
```

Requires host `aapt` and `apksigner` (or Android build-tools on `PATH` / `ANDROID_HOME`).

## Schedule

Default cron: **Saturday 02:00 UTC**. Full rebuild every run; packages that fail download/patch are skipped.
