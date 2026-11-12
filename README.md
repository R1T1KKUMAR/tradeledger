# TradeLedger — P&L Journal

A daily trading profit-and-loss journal, kept in ink. A single-page web app with a ledger-sheet design: pale-green paper in day mode, ink-on-slate at night. Built for Indian traders (₹, NIFTY/BANKNIFTY/crypto USDT pairs), but it works for any instrument in any currency spirit.

**Live: <https://r1t1kkumar.github.io/tradeledger/>**

## What it does

- **Journal** — one entry per trading day (gross P&L, charges, instrument, note). Net is calculated for you. Net P&L verdict with a rotated rubber-stamp, six-cell summary strip (win rate, best/worst day, average, current streak, max drawdown), a 364-day heatmap of green/red squares, an SVG equity curve with hover tooltip and dashed peak line, a daily-bars chart, dotted-leader breakdown (profit factor, charges paid), and a month-by-month review ledger.
- **My Data** — searchable, sortable, paginated table of every stored entry; filter by instrument, result, or date range; edit or delete any row (with Undo); instrument chips; CSV + JSON export/import.
- **Cloud sync (optional)** — create a free account (email + password, or Continue with Google) and your ledger syncs to a private per-user Firestore database: live multi-device sync, newer-wins conflict resolution, offline-first with automatic catch-up.
- **Backups** — CSV for spreadsheets, JSON full backup/restore, backup-age reminder.
- **PWA** — installable on phone/desktop, works offline via service worker.

Your data stays in your browser until you create an account; with an account, it lives in your own private cloud document collection that only you can read.

## Run locally

Any static server works:

```bash
python3 -m http.server 8000
# or: npx serve
```

Open <http://localhost:8000>. (Cloud sync requires the Firebase config below; everything else works immediately.)

## Firebase setup (~5 minutes, free)

**Already configured for the live site** — the production project `tradeledger-app` is wired up: Firestore (asia-south1), the per-user rules below, and both sign-in providers are live. This section remains for anyone forking the app and using their own backend.

The app is fully functional without this — accounts and sync are the only parts that need it.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** (any name, e.g. "TradeLedger"). Analytics optional.
2. **Build → Authentication → Get started** → enable **Email/Password** and **Google** as sign-in providers.
3. **Build → Firestore Database → Create database** → start in **production mode**; pick a regional location.
4. **Project settings (⚙ gear) → Your apps → Web (`</>`)** → register the app as "TradeLedger" (no hosting needed) → copy the `firebaseConfig` values.
5. Paste them into `index.html`:

   ```js
   const FIREBASE_CONFIG={apiKey:'AIza…',authDomain:'…',projectId:'…',appId:'…'};
   ```

6. **Authentication → Settings → Authorized domains → Add domain**: `r1t1kkumar.github.io` (plus `localhost` for local dev).
7. **Firestore → Rules** → paste the contents of [`firestore.rules`](firestore.rules) from this repo → **Publish**.

That's it — the **Create account** button in My Data → Cloud sync comes alive, and each user gets a private `users/{uid}/entries/{date}` collection.

### Security model

`firestore.rules` enforces that a signed-in user can only read and write documents under `users/{their-own-uid}/entries/`. There is no admin API in the client, no shared data, no other collections.

## Sync model

- Local-first: every edit writes to `localStorage` immediately and marks the touched date dirty.
- Dirty dates flush to Firestore in a batch ~2.5 s after your last change (and on unload/online).
- On sign-in, local entries are merged into the cloud (per-date, newer `updatedAt` wins; legacy entries without a timestamp are uploaded and stamped).
- A live `onSnapshot` listener keeps every signed-in device in sync; remote changes only apply when they're newer or the content actually differs.
- Deleting a date deletes the cloud document; deleting your account data is as simple as deleting the entries (or wipe-all with Undo).

## Tech

Single `index.html` (no framework, no build step), vanilla JS, Newsreader + IBM Plex Sans from Google Fonts, Firebase compat SDK 10.12.2 (app/auth/firestore) from Google's CDN, hand-rolled SVG charts, hand-generated PNG icons. PWA via `manifest.webmanifest` + `sw.js`.

## License

MIT — see [LICENSE](LICENSE).
