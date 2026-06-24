# obsidian-config

Obsidian vault skeleton configuration for the pilab k3s deployment.

This repository seeds the `.obsidian/` directory (plugins, themes, snippets, base
settings) onto the vault PVC on first pod start. Vault *content* is managed entirely
by Obsidian Sync and is never committed here.

## What's tracked

| Path | Purpose |
|---|---|
| `.obsidian/app.json` | Core Obsidian preferences |
| `.obsidian/hotkeys.json` | Keyboard shortcuts |
| `.obsidian/community-plugins.json` | Enabled community plugins list |
| `.obsidian/plugins/` | Bundled community plugin data |
| `.obsidian/themes/` | Custom themes |
| `.obsidian/snippets/` | CSS snippets |

## What's excluded (via .gitignore)

- `workspace.json` / `workspace-mobile.json` — open pane state, machine-specific
- `cache` — local search/render cache
- `sync-state` — Obsidian Sync internal bookkeeping

## Obsidian Sync: one-time manual login

Obsidian Sync authentication **must be performed manually** through the noVNC
session on first run, and must be re-done if the vault PVC is ever destroyed.

**Steps:**
1. Open the Obsidian noVNC URL in a browser (behind Cloudflare Access).
2. Inside the Obsidian desktop, open **Settings → Sync**.
3. Log in with your Obsidian account and connect the remote vault.
4. Wait for the initial sync to complete.

Auth state persists in the PVC at `/config` and survives pod restarts.
It does **not** persist if the PVC is deleted. If you recreate the PVC, repeat
the steps above.

## How the init container uses this repo

The `seed-obsidian-config` init container in the Obsidian deployment does the
following on every pod start:

```sh
if [ ! -d /config/.obsidian ]; then
    git clone --depth=1 https://github.com/gorttman/obsidian-config.git /tmp/obs-cfg
    cp -r /tmp/obs-cfg/.obsidian /config/.obsidian
fi
```

The check is **idempotent**: once `.obsidian/` exists on the PVC it is never
overwritten. To reset config, delete `.obsidian/` on the PVC and restart the pod.

## Repository visibility

This repository must be **public** so the init container can clone it without
credentials. It contains no secrets.
