# AGENTS.md

Single-file portfolio bundle. No build system, package manager, or tests.

## Repo shape

- `saniya_portfolio_main.html` — the entire app: a self-unpacking "bundler" wrapper that inlines all JS (React + ReactDOM + Babel standalone), fonts (woff2), and images as base64/gzip blobs in `__bundler/manifest`, then swaps the real template (in `__bundler/template`) into the DOM at runtime via `DOMParser` + script re-hydration.
- `assets/` — loose copies of project preview images and the profile photo (also embedded in the bundle). Not referenced by the HTML at runtime.

## How to run

Open `saniya_portfolio_main.html` directly in a browser. No server required, but `file://` works because assets are blob URLs from the embedded manifest. JS must be enabled (the page shows a `<noscript>` fallback otherwise).

## Editing rules (important — easy to break)

- The page is React written as `text/babel` JSX, transformed in-browser by Babel standalone at runtime. There is no compile step.
- Three inline `<script>` payloads must stay byte-intact and in order: `__bundler/manifest` (JSON of `{uuid: {mime, compressed, data}}`), `__bundler/ext_resources` (JSON array), `__bundler/template` (JSON string of the real HTML doc). Do not reformat these — the wrapper does exact-string UUID substitution into the template.
- The wrapper strips `integrity=` and `crossorigin=` from the template before injecting (blob URLs from `file://` have null origin; SRI/CORS would break). If you add external scripts to the template, expect those attributes to be removed.
- `text/babel`/`text/jsx` scripts with a `src` are fetched and inlined by the wrapper (blob:null fetch is silently dropped on `file://`). Prefer inline scripts for new code.
- Editing the actual React/JSX source means editing the JSON string inside `__bundler/template`. Escape `</script>` as `<\/script>` and keep it valid JSON (newlines as `\n`, quotes escaped). Avoid naive string edits of the raw file — parse the JSON, mutate, re-serialize.
- Asset UUIDs in the template (`url("<uuid>")`, `src="<uuid>"`) map to entries in the manifest. To add a new asset you must add a manifest entry and reference its UUID in the template.

## No tooling

- No `package.json`, no lint/typecheck/test commands, no CI. Don't run `npm`/`yarn` — there's nothing to install.
- Verify changes by reloading the page in a browser and watching the console (`[bundler]` / `[bundle]` prefixed errors come from the unpacker).