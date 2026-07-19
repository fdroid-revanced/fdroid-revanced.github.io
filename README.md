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

1. Install Java, Go, `fdroid`, `aapt`, `apksigner` via mise
2. `GOPROXY=direct go run github.com/lucasew/revancedbot/cmd/revancedbot@main run .` with secret `REVANCEDBOT_SIGNING`
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
4. Run workflow **Enqueue F-Droid build** to stack work (`count` = how many full builds). Optional **smoke**. The **Build F-Droid repo** worker drains the queue one run at a time (no runner waiters).

## Local rebuild

```bash
mise install   # java, go, fdroidserver
export REVANCEDBOT_SIGNING='…'   # same secret
mise run build-repo   # go run …@main under the hood (GOPROXY=direct)
```

Requires host `aapt` and `apksigner` (or Android build-tools on `PATH` / `ANDROID_HOME`).

## Schedule

Default cron: **every 8 hours** (`0 */8 * * *` UTC) via **Enqueue F-Droid build** (+1 to the queue). Workers run serially; spam enqueue to stack as many builds as you want without losing them. Soft-skips on download/patch failure still apply.
