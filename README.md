# pt-smart-board-cdn

CDN-published body for the **PT-Smart-Board** Packet Tracer activity overlay.

The companion PKA (`2.2-CloudShell.pka`) is a thin shell that fetches the
current body from this repo at runtime via [jsDelivr][jsd]. To roll out a
fix or a feature, push a new version here and the next time anyone opens
the activity in PT they'll get the updated body — no PKA re-issue
needed.

[jsd]: https://www.jsdelivr.com/

## Layout

```
manifest.json              ← shell reads this first
dist/
└── board-vX.Y.Z.html      ← one immutable file per release
```

`manifest.json`:

```json
{
  "currentVersion": "2.0.0",
  "versions": {
    "2.0.0": "dist/board-v2.0.0.html"
  },
  "updatedAt": "..."
}
```

The shell:

1. Fetches `https://cdn.jsdelivr.net/gh/ioeacademy/pt-smart-board-cdn@latest/manifest.json`
2. Reads `currentVersion`, looks up its path in `versions`
3. Fetches that path from the same `@latest` (jsDelivr handles the version pinning)
4. Loads the HTML into an iframe via `srcdoc`

If any fetch fails (offline, CDN down, repo deleted), the shell falls back to a
last-known-good copy embedded inside the PKA itself — the activity always
opens, just possibly at an older version.

## Publishing a new release

From the main `PT-Smart-Board-V2.0` repo:

```bash
# 1. Bump package.json version
# 2. Build the body
node build.mjs
# 3. Copy + commit to this repo (use scripts/publish-cdn.mjs)
node scripts/publish-cdn.mjs
```

The publish script copies `dist/Handsfreetopology.html` to
`dist/board-v<version>.html` here, updates `manifest.json` to point at
it, and commits + pushes. jsDelivr picks up changes within ~10 minutes
on `@latest`; for instant rollout use a tagged URL.

## Bandwidth

jsDelivr is free and unmetered for public repos. Single-file pulls are
~400 KB — even with thousands of opens per day this stays well within
the soft fair-use limits.
