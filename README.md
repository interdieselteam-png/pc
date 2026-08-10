# Petty Cash Ledger

A small offline-first PWA for tracking petty cash in and out. Data is stored
locally in the browser (`localStorage`) — nothing is sent to a server.

## Run locally

Serve the folder with any static file server (service workers require
`http://` or `https://`, not `file://`):

```
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## Deploy to GitHub Pages

This repo includes a GitHub Actions workflow
(`.github/workflows/deploy-pages.yml`) that deploys the site automatically
on every push to `main`.

To enable it:

1. Go to the repo's **Settings → Pages**.
2. Under **Build and deployment → Source**, choose **GitHub Actions**.
3. Push/merge to `main` — the workflow will build and publish the site.
4. The app will be available at `https://<owner>.github.io/<repo>/`.

## PWA

- `manifest.webmanifest` — app metadata, icons, theme colors.
- `sw.js` — service worker that caches the app shell for offline use and
  updates the cache in the background when a fresh copy is available.
- `icons/` — app icons (192/512, including maskable variants).

Because the app is served from a repo subpath on GitHub Pages
(`/<repo>/`), all asset paths are relative so it works both locally and
once deployed.
