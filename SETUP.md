# GA Hot Leads CRM — Setup Guide (for Luca)

One-time setup, about 10 minutes. The app file is `index.html` (single file, no build step).

## Step 1 — Share both sheets (2 min)

In Google Drive, for **GA ZOMBIE LEADS** and **GA STACKED LEADS**:

1. Right-click → **Share** → **General access** → **Anyone with the link**
2. Set the role to **Viewer**
3. Done. The app only reads these sheets; it can never write to them.

## Step 2 — Create the Sheets API key (3 min)

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → select (or create) a project
2. **APIs & Services → Library** → search **Google Sheets API** → **Enable**
3. **APIs & Services → Credentials** → **Create Credentials → API key**
4. Click the new key → **API restrictions** → **Restrict key** → check **Google Sheets API** *and* **Google Drive API** → Save
   - The Drive API powers the property-photo thumbnails (see "Photos" below). If you skip it, everything else works — photos just won't load.

5. Open `index.html`, find `CONFIG` near the top of the script, and paste the key:

```js
const CONFIG = {
  SHEETS_API_KEY: "PASTE_IT_HERE",
  ...
};
```

The two sheet IDs are already filled in. Nothing else in the file needs editing.

## Step 3 — Enable Google sign-in (2 min, required for the lock below)

The app uses your existing Firebase project (`rgr-crm-941ab`) — no new project needed.

1. [Firebase Console](https://console.firebase.google.com/) → **rgr-crm-941ab** → **Authentication → Sign-in method**
2. Enable the **Google** provider → Save

Without this, the "Sign in with Google" button will fail and the app can't load your private data.

## Step 4 — Lock down Firestore rules (3 min, do this BEFORE real data goes in)

1. Firebase Console → **rgr-crm-941ab** → **Firestore Database → Rules**
2. Merge the blocks from `firestore.rules` (in this folder) into your existing rules → **Publish**
3. **Critical:** remove or tighten any broad `allow read, write: if true` rule. Firestore grants access if *any* matching rule allows it, so a leftover catch-all silently overrides the lock.

Result: the three Hot Leads collections (`zombieLeads`, `stackedLeads`, `hotLeadsActivityLog`) are **locked to your Google account only** — reads and writes. The app only subscribes to your statuses, notes, phones, tasks, family trees, manual leads, and the activity report while you're signed in; signing out clears them from the screen. (The lead list itself still loads from the link-shared sheets, which are public by design — Firestore holds only your private working layer.)

## Step 5 — Deploy to GitHub Pages (like your probates CRM)

1. Create a new repo (e.g. `lucarebuffi-rgr/ga-hot-leads-crm`), add `index.html` at the root
2. Repo **Settings → Pages** → Deploy from branch → `main` → `/ (root)` → Save
3. Your app will be live at `https://lucarebuffi-rgr.github.io/ga-hot-leads-crm/`

## How it works (the 30-second version)

- **Google Sheets = the master database.** The nightly pipeline keeps writing leads there exactly as today. The app reads both sheets live, matching columns **by header label** (never by position), so column reorders can't break it.
- **Firestore = the interactive layer.** Statuses, call log, notes, tasks, phone numbers, and family trees are stored per-lead in Firestore, keyed by normalized address (`zombieLeads/{…}`, `stackedLeads/{…}`). Nothing ever writes back to the sheets.
- **Report mode:** open the app with `?report=1` (there's a Report button in the header). Pick Today / Last 7 Days / Last 30 Days / All Time, and see activity grouped **by acquisition director** — calls logged, stage changes, dial clicks, totals — plus the latest 100 events.

## Ported from your probates CRM (Sept 2026 changes)

Everything you added to the main CRM is in here too:

- **Acquisition directors** — assign each lead to you, Jalen, Salena, Ricky, or Tony from the expanded card; director badge on the card, filter leads by director, and the report attributes every call/stage change/dial click to the lead's director.
- **Agenda** — 📅 Today's Tasks / Tomorrow's Tasks / Dormant Tasks (overdue) buttons with live counts; one card per lead, filterable by director.
- **❗ No Task badge** — leads in Qualified/Follow Ups/Under Contract with no open task get flagged on the card.
- **+ Add Lead** — manually add a lead (phone + address required). It's stored in Firestore only — the sheets stay read-only, so a manual lead never appears in the sheet. Deduped by address.
- **Bulk delete** — checkboxes on every card, Select all, Delete Selected. Manually added leads are removed entirely; sheet-backed leads are *hidden* from the CRM (sheet rows untouched) — use "Show hidden" to bring them back.
- **Added-date filter** — filter leads by the date they entered the CRM (Date Flagged for zombies, earliest motivation date for stacked, add date for manual leads).
- **Deal numbers** — Lead Value, Asking Price, LAO, MAO money fields on every expanded card.
- **Photos** — paste a Google Drive folder link on any Qualified/Follow Ups/Under Contract/Closed lead and get a photo thumbnail carousel (folder must be shared "Anyone with the link").
- **Dial tracking** — tapping the 📞 call link logs a "dial click" in the activity feed and report.

## If something looks wrong

- **"Sheets blocked (403)"** in the header → the sheet isn't link-shared yet (Step 1).
- **"Bad API key"** → the key wasn't pasted into `CONFIG`, or the Sheets API isn't enabled on it (Step 2).
- **Can't save a status/note** → you're not signed in (Step 3), or the rules aren't published (Step 4).
