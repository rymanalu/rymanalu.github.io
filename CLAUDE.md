# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal portfolio site for Roni Yusuf, served by GitHub Pages as a user site at
`https://rymanalu.github.io`.

There is no CI, no build step, no `CNAME`, and no `.github/` directory — Pages serves the repo
root of `master` verbatim, and all history lives on `master`. **Pushing to `master` is the
deploy.** There are no tests, linters, formatters, or package manifests; nothing to install and
nothing to run.

## The site is one self-contained file

`index.html` is the entire site: markup, a small `<style>` block, and inline styles on every
element. It has **no `<script>` tags at all** and loads exactly two things over the network —
Google Fonts (Bitter + Libre Franklin) and nothing else. Editing it is the whole workflow; what
you save is what ships.

It has no `id` attributes and no in-page navigation, so there is nothing to keep in sync between
a menu and its sections.

The page was generated from the Claude Design project **"rymanalu.github.io rebuild"**
(`3857e527-86e7-42f3-a9c3-97385609e821`), which is the design source of truth. Substantial visual
changes are best made there and re-imported via the `DesignSync` MCP tool (`get_file` on
`index.html`); small copy edits are fine to make directly here. Re-importing overwrites the whole
file, so re-apply the two local deviations listed below afterwards.

### Two deliberate deviations from the design source

Both will be silently reverted by a naive re-import — re-apply them:

1. **The resume link is `href="Roni%20Yusuf.pdf"`, not `Roni_Yusuf.pdf`.** The file at the repo
   root is `Roni Yusuf.pdf` — the space is load-bearing, because `/Roni Yusuf.pdf` is the URL
   that has been public for years. The `download="Roni_Yusuf.pdf"` attribute gives visitors the
   underscored filename anyway, so the encoded href costs nothing. Renaming the file would break
   any existing external link for no user-visible gain.
2. **`<link rel="icon" href="favicon.ico">` is added back**; the design omits it.

## `favicon.ico` at the repo root is load-bearing beyond `index.html`

`lot-calculator/index.html` requests **root-relative** `/favicon.ico`. Deleting or moving the
root favicon breaks the sub-app's icon, not just the main page's. (`loketh/` uses its own
`./favicon.ico` under its `<base href="/loketh/">` and is unaffected.)

## Verifying a change

There is no test suite, so verify visually and structurally. A headless Chromium ships with the
Playwright browser cache at:

```
~/.cache/ms-playwright/chromium-1228/chrome-linux64/chrome
```

Screenshots via `--headless --screenshot` are fine for checking appearance. For anything that
depends on **viewport width** — this page uses `clamp()` with `vw` units throughout — do not use
`--window-size` below ~500px: Chrome floors the window at ~485px on Linux and silently reports
that width instead, so narrow-viewport results are wrong rather than absent. Drive
`Emulation.setDeviceMetricsOverride` over the DevTools Protocol instead. Node's global
`WebSocket` is enough to do this with no packages installed.

## Sub-apps: `loketh/` and `lot-calculator/`

Both are **prebuilt Create React App bundles copied in from separate repos**. There is no source,
no `package.json`, and no build config for either one here — only hashed `static/js/*.chunk.js`
bundles, a `service-worker.js`, and a `precache-manifest.*.js`.

- `loketh/` — Ethereum event-ticketing smart-contract demo. Pins `<base href="/loketh/">`.
- `lot-calculator/` — position-sizing calculator for share purchases.

Never hand-edit the hashed bundles or precache manifests; the filename hashes and the service
worker's cached-URL list would desync, serving stale assets to returning visitors. To update
either app, rebuild it in its upstream project and copy the resulting `build/` output over the
directory wholesale (this is what the "Update loketh" commits in the history are).

Neither app is linked from `index.html` — both are reached only by direct URL.

## History: what used to be here

Through Aug 2026 the site was Ryan Fitzgerald's `devportfolio-template` v1.2.2, with a
gulp + `node-sass` pipeline whose generated output (`css/styles.css`, `js/scripts.min.js`) was
committed alongside its sources. The rebuild made all of it unreachable, so the pipeline
(`gulpfile.js`, `package.json`, `scss/`), the generated and vendored assets (`css/`, `js/`,
`libs/font-awesome/`), and the template's images were deleted. Nothing references them; do not
resurrect them to make a styling change — edit `index.html` directly.

Google Analytics went with it. `js/analytics.min.js` carried a **Universal Analytics** property
(`UA-79154045-2`); Google stopped processing data into UA properties in July 2023, so it had been
sending beacons nowhere for years. The site now has **no analytics and no third-party scripts**.
Adding measurement back means a fresh GA4 property, not restoring the old snippet.
