# app — Firefox in WebAssembly (Gecko)

A complete browser that runs inside a tab: the Gecko engine compiled to WebAssembly, with
the real Firefox interface. Nothing here calls a server of its own apart from the engine
payloads on the upstream Puter CDN.

## Files

| File | Size | Purpose |
|---|---|---|
| `index.html` | 4,201,149 B | the whole app, engine loader included |
| `coi-serviceworker.js` | 6,373 B | gives the page cross-origin isolation so SharedArrayBuffer exists |
| `manifest.json` | 655 B | PWA install metadata, relative paths only |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png` | | icons referenced by the manifest |
| `.nojekyll` | 0 B | stops Jekyll from processing the site |

## Why the service worker is required

Gecko needs SharedArrayBuffer, and browsers only expose it to a cross-origin isolated
document, which means the host must send these two headers on the document:

    Cross-Origin-Opener-Policy: same-origin
    Cross-Origin-Embedder-Policy: require-corp

GitHub Pages cannot send custom headers. `coi-serviceworker.js` (v0.1.7 by Guido Zuidhof,
MIT licence, patched so cross-origin engine downloads bypass the worker) registers a
service worker that adds those headers to this origin's own responses, which is enough.
Without it the engine loads, then stalls, and no page will open — that was the original
failure.

## Deploy

1. Create a public repository named `app`.
2. Upload every file to the repository root.
3. Settings -> Pages -> Deploy from a branch -> `main` / `(root)` -> Save.
4. Open https://mccraekaiser48-cell.github.io/app/

The first visit registers the worker and reloads itself once (twice if it has to fall back
from `credentialless` to `require-corp`). After that, check the console:

    crossOriginIsolated        // true
    typeof SharedArrayBuffer   // "function"

If isolation is still false, the worker is not controlling the page. Open DevTools ->
Application -> Service Workers and confirm its scope is `/app/`.

## Networking

Pages inside the browser are fetched through a WISP relay, preset to
`wss://wisp.mercurywork.shop/` and changeable under the app's Advanced options.

## Remove the worker

    navigator.serviceWorker.getRegistrations().then(r => r.forEach(x => x.unregister()))

then reload twice. Deleting `coi-serviceworker.js` before unregistering leaves a broken
worker behind.
