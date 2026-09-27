# NHL Breakout Model

A client-side fantasy-hockey tool: load MoneyPuck skater CSVs, compute
breakout-candidate rankings, backtest the model against real outcomes,
and chart player career trajectories (pre-NHL leagues translated to
NHL-equivalent rates alongside actual NHL production).

Everything runs in the browser. There is no backend and no server-side
database - all loaded data lives in your browser's IndexedDB, scoped to
whichever URL you're viewing it from (see **Data storage** below).

## Running it locally

No build step. Either:

- Open `index.html` directly in a browser, or
- Serve the folder so `fetch`/module behavior matches production exactly:
  ```
  python3 -m http.server 8000
  ```
  then visit `http://localhost:8000`.

## Deploying

Pick one. Both are free for a project this size.

### Option A: GitHub Pages (simplest, no extra account needed)

1. Push this repo to GitHub.
2. Repo Settings -> Pages -> Source: **GitHub Actions**.
3. Push to `main` - `.github/workflows/deploy-gh-pages.yml` builds and
   deploys automatically. Your site will be at
   `https://<username>.github.io/<repo-name>/`.

No secrets or config edits required for this path.

### Option B: Firebase Hosting

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com) if you don't have one.
2. Edit `.firebaserc` - replace `REPLACE-WITH-YOUR-FIREBASE-PROJECT-ID` with your actual project ID.
3. Deploy by hand:
   ```
   npm install -g firebase-tools
   firebase login
   firebase deploy
   ```
   Your site will be at `https://<project-id>.web.app`.
4. **Optional - auto-deploy on push:** edit `.github/workflows/deploy-firebase.yml`,
   set `projectId` to match `.firebaserc`, then generate a service account key
   (`firebase init hosting:github` will do this for you and add the secret
   automatically - easier than doing it by hand) and add it as the
   `FIREBASE_SERVICE_ACCOUNT` repo secret.

### Running both at once

You can deploy to GitHub Pages and Firebase Hosting simultaneously - they're
just two different static hosts serving the same files. See **Data storage**
below for why that means two separate, unsynced copies of your loaded data.

## Data storage

The app stores everything in the browser's IndexedDB: seasons you load,
prospect stats you enter, league coefficients you edit. This is:

- **Per-browser, per-origin.** Data saved while visiting the site at one
  URL (e.g. your GitHub Pages URL) will *not* appear if you open a
  *different* URL (e.g. a Firebase URL, or `localhost`), even though it's
  the same code. Each origin gets its own isolated storage.
- **Not synced across devices or browsers.** Loading a season on your
  laptop doesn't make it appear on your phone.
- **Persistent** across visits/tabs on the same browser + origin, until
  you clear site data or browser storage.

If you want this data to follow you across devices, keep your source
MoneyPuck CSVs as the real backup and re-load them wherever you're
working - or say so and a real backend (Firestore, since you're already
on Firebase) is a natural next step.

## What's in the app

- **Load Data** - upload a MoneyPuck skaters CSV, tag it with a season number.
- **Seasons** - see what's loaded, delete a season.
- **Rankings** - breakout scores with adjustable signal weights and a
  minimum-games-played floor (small-sample call-ups otherwise dominate
  the list on noise alone).
- **Career Chart** - full player trajectories, pre-NHL seasons translated
  to NHL-equivalent points-per-game via editable league coefficients.
- **Prospects & Leagues** - NHLe coefficient table (seeded with published
  community estimates - treat as a starting point, not ground truth) and
  a form for entering junior/college/European career stats.
- **Backtest** - score a past season and check the predictions against
  what actually happened, with rank correlation and precision@K.

Player names are disambiguated by MoneyPuck's own `playerId` when
present, so two real players sharing a name (this happens - two current
NHL players are both named "Elias Pettersson") don't get merged into one
identity. This disambiguation is per-file-load; it isn't (yet) persisted
as a stable identity across separately loaded seasons.

## Tech

Single HTML file. [PapaParse](https://www.papaparse.com/) for CSV parsing,
[Chart.js](https://www.chartjs.org/) for the career chart, both loaded from
cdnjs. No build tooling, no npm dependencies to install for the app itself
(only for the Firebase CLI, if you use that deploy path).
