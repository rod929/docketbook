# Deploying Docketbook

Docketbook is a single file — `index.html` — served by GitHub Pages at
https://rod929.github.io/docketbook/.

## How a change goes live
1. The change is committed to the **`main`** branch (Claude Code can push here directly).
2. GitHub Pages rebuilds automatically. This usually takes **1–3 minutes**.
3. Open the live site and do a **hard refresh** (Ctrl/Cmd + Shift + R) or use an
   **incognito / private window**. Browsers cache aggressively, so a normal refresh
   can keep showing the old version.

## Testing notes (important)
- Always test on the **live https:// address**, never the local `file://` copy —
  logins and saved data do not work from `file://`.
- If a change doesn't show up: wait a couple of minutes for the Pages rebuild, then
  hard-refresh. If it still doesn't show, check that the file on GitHub actually
  contains the change before assuming something is broken.

## Branches
- **`main`** — what GitHub Pages serves (the live site).
- **`claude/docketbook-deploy-rkj5ru`** — Claude Code's working branch, used for
  changes that should be reviewed before they go live.
