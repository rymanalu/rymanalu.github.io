# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal portfolio site for Roni Yusuf, served by GitHub Pages as a user site at
`https://rymanalu.github.io`. Built on Ryan Fitzgerald's `devportfolio-template` v1.2.2 (MIT).

There is no CI, no `CNAME`, and no `.github/` directory — Pages serves the repo root of `master`
verbatim, and all history lives on `master`. **Pushing to `master` is the deploy.**

## Generated files are committed, and they are what ships

Because Pages runs no build step, every artifact the browser loads is checked in. `index.html`
loads only generated or vendored files — never the sources:

| Browser loads              | Source              | Built by                     |
| -------------------------- | ------------------- | ---------------------------- |
| `css/styles.css`           | `scss/styles.scss`  | `gulp styles`                |
| `js/scripts.min.js`        | `js/scripts.js`     | `gulp scripts`               |
| `js/analytics.min.js`      | `js/analytics.js`   | **nothing — hand-maintained** |
| `css/bootstrap.min.css`    | —                   | vendored, do not edit        |
| `libs/font-awesome/`       | —                   | vendored, do not edit        |
| jQuery 1.12.4              | —                   | Google CDN, loaded first     |

Editing `scss/styles.scss` or `js/scripts.js` alone has **no effect on the live site** — the
matching generated file must be rebuilt and committed alongside it.

`js/analytics.js` sits outside the gulp pipeline entirely (`gulpfile.js` wires up only
`js/scripts.js` and `scss/styles.scss`). The two files are currently in sync — `analytics.min.js`
is a faithful minification that inlines the tracker URL — but there is **no command that
regenerates it**. Changing the GA snippet (currently property `UA-79154045-2`) means editing both
files, minifying by hand or adding a gulp task for it.

## Build commands

The lockfile is npm's `package-lock.json`. There are no tests, linters, or type checks.

```sh
pnpm install            # or `npm ci`; pnpm also writes an untracked pnpm-lock.yaml
pnpm run watch          # gulp watch — rebuilds either source on change
pnpm exec gulp styles   # scss/styles.scss -> css/styles.css (compressed)
pnpm exec gulp scripts  # js/scripts.js -> js/scripts.min.js (babel preset-env + uglify)
```

**The three gulp commands do not currently run on modern Node — see below.** Only `pnpm install`
succeeds.

### The build does not run on modern Node

`gulp-sass@4` depends on `node-sass@4.14.1`, whose prebuilt bindings stop at Node 14. Verified on
Node 24 — installation appears to succeed (pnpm skips the native build script by default), then
loading the module throws:

```text
Node Sass does not yet support your current environment: Linux 64-bit with Unsupported runtime (137)
```

`gulpfile.js` requires `gulp-sass` at the **top level**, so gulp cannot finish loading the gulpfile
— **every task fails, not just `styles`.** The `scripts` task is pure JS (babel + uglify) and would
work fine on its own; it is unreachable rather than broken. To run any build, pick one:

- Run the build under Node 14 via nvm — smallest change, keeps the pipeline intact.
- Migrate `gulp-sass`/`node-sass` to `sass` (dart-sass). The migration surface is small:
  `scss/styles.scss` is a single self-contained 1001-line file with zero `@import`s, and the only
  color functions used are `darken()`, `lighten()`, and `rgba()`.
- Last resort, CSS only: hand-edit `css/styles.css` and mirror the change back into
  `scss/styles.scss`. The two drift silently, so avoid this unless the change is trivial.
  There is no equivalent escape hatch for JS — `js/scripts.min.js` has to be regenerated.

## Editing site content

All content is hand-authored HTML in `index.html` (364 lines, single page). The `#menu` list at
the top drives in-page navigation; each entry must match a section `id` further down.

The experience timeline is **data-driven at runtime**. Each direct child `<div data-date="...">`
of `#experience-timeline` is wrapped by `js/scripts.js` into the
`.vtimeline-point` / `.vtimeline-block` / `.vtimeline-content` structure, with `data-date` emitted
as a `.vtimeline-date` span and a map-marker icon prepended. **Adding a job means adding one
`data-date` div — no JS or CSS change is needed.**

The resume is `Roni Yusuf.pdf` at the repo root, linked from the lead section as `/Roni Yusuf.pdf`.
The space in the filename is load-bearing; renaming the file requires updating that link.

Theme colors are SCSS variables near the top of `scss/styles.scss` (`$base-color`, `$background`,
`$heading`, `$text`, …) — change them there, not in the compiled CSS.

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

## Known pre-existing defects

Recorded so they aren't mistaken for new breakage. Fix only if asked.

- **The `#certifications` nav link is broken.** `#menu` links to `#certifications`, but no element
  carries that id — the Certifications section reuses `id="education"`, so `index.html` has two
  `id="education"` divs. The nav handler in `scripts.js` calls `$(heading).offset().top`, and
  jQuery's `.offset()` returns `undefined` for an empty set, so clicking "Certifications" throws a
  `TypeError` and scrolls nowhere. The fix is renaming the second `id="education"` (the one headed
  "Certifications") to `id="certifications"`.
- **Two competing year stamps.** `scripts.js` fills `#current-year`, which does not exist in the
  markup. The footer's `#currentYear` is populated instead by a separate inline `<script>` at the
  bottom of `index.html`.
- **Dead project handlers.** `scripts.js` binds `#view-more-projects` and `#more-projects`; neither
  exists in the markup. They are template leftovers — the site has no projects section.
