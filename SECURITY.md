# Security

Dustline Rally is a static, client-side game: no backend, no accounts, no analytics, and no data leaves the browser. The only stored data is two settings in `localStorage` (`dl.assist`, `dl.line`).

## What is in place

- A Content-Security-Policy `<meta>` limits scripts to the inline game code and the pinned three.js build on unpkg, limits styles to inline and Google Fonts, and blocks other network connections, `<base>` and form submission.
- three.js is pinned to an exact version and loaded with Subresource Integrity (import map plus `modulepreload` links).
- `referrer: no-referrer`, so the page does not send its address to the CDNs.
- The results table is built with DOM methods, not `innerHTML`.

## Known limits

GitHub Pages cannot send custom HTTP headers, so `frame-ancestors`, HSTS and `X-Content-Type-Options` cannot be set. The Google Fonts stylesheet cannot carry an integrity hash. `window.__dbg` is a debug hook used by the tests; it is not a security boundary.

## Reporting a problem

Please use GitHub's private vulnerability reporting for this repository (Security tab, "Report a vulnerability"), or open an issue if the problem is not sensitive.
