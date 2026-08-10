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

## Google Sheets sync (optional)

The app can mirror entries to a Google Sheet as a backup. `localStorage`
stays the source of truth; every add/edit/delete overwrites the sheet's
data range with the current entry list. Receipt photos are never sent —
only the text fields (date, type, amount, category/source, party, note).

This is entirely client-side (no backend), so setup means creating your
own OAuth Client ID and API key in Google Cloud Console and pasting them
into `index.html`.

### 1. Create a Google Cloud project

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and
   create a new project (or reuse one).
2. Under **APIs & Services → Library**, enable:
   - **Google Sheets API**
   - **Google Picker API**

### 2. Configure the OAuth consent screen

1. **APIs & Services → OAuth consent screen**.
2. Choose **External**, fill in the required fields (app name, your email).
3. Add your own Google account under **Test users** (while the app is in
   "Testing" mode, only test users can sign in — that's fine for personal
   use; you don't need to submit for verification).

### 3. Create credentials

1. **APIs & Services → Credentials → Create Credentials → OAuth client ID**.
   - Application type: **Web application**.
   - Authorized JavaScript origins: `https://<owner>.github.io` (and
     `http://localhost:8080` if you want it to work locally too).
   - Copy the generated **Client ID**.
2. **Create Credentials → API key**.
   - Restrict it to the Sheets API and Picker API.
   - Copy the generated **API key**.

### 4. Wire it into the app

In `index.html`, find the config block near the top of the `<script>` tag
and replace the placeholders:

```js
const GOOGLE_CLIENT_ID = 'YOUR_CLIENT_ID.apps.googleusercontent.com';
const GOOGLE_API_KEY = 'YOUR_API_KEY';
```

Commit, push, and redeploy. A "Connect" button appears under the header —
clicking it signs the visitor into their own Google account (each user
authenticates individually; the Client ID just identifies the app to
Google) and lets them pick or create the spreadsheet to sync to.

### Security notes

- The Client ID and API key are meant to be public in browser apps —
  visible in page source is expected. Security comes from restricting the
  Client ID to your GitHub Pages origin and the API key to specific APIs,
  not from hiding them.
- The app requests the `drive.file` scope, not full Drive access — it can
  only see the one spreadsheet a user explicitly picks, not their whole
  Drive.
- The access token lives only in memory (a JS variable), never in
  `localStorage`. It expires after about an hour and isn't persisted, so
  each new session requires signing in again.
- Users can revoke access at any time from
  [myaccount.google.com/permissions](https://myaccount.google.com/permissions).
