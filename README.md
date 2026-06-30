# BvB Youth Soccer — Team Manager

A complete team management web app: roster, benchmarks, progress reports, PDF exports, QR-code tryout sign-up, password protection, and free cloud sync. Works as a website **and** installs as an app on iPhone/Android home screens (Progressive Web App).

## 🚀 Deploy to GitHub Pages (free hosting)

### Step 1 — Create a GitHub repository
1. Go to [github.com/new](https://github.com/new)
2. Name it something like `bvb-team-manager`
3. Set it to **Public** (required for free GitHub Pages)
4. Click **Create repository**

### Step 2 — Upload these files
Upload all files in this folder, keeping the same structure:

```
bvb-team-manager/
├── index.html
├── manifest.json
├── service-worker.js
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

**Easiest way:** On your repo page, click **"uploading an existing file"**, drag everything in, and commit.

### Step 3 — Turn on GitHub Pages
1. In your repo, go to **Settings → Pages**
2. Source: **Deploy from a branch** → Branch: `main`, folder: `/ (root)`
3. **Save**, wait ~1 minute, refresh — your live URL appears:
   ```
   https://YOUR-USERNAME.github.io/bvb-team-manager/
   ```

---

## 🔒 Password protection

The app is locked behind a single shared team password before anyone can see the roster, reports, etc.

**Default password:** `test`

### To change the password:
1. Open `index.html` in a text editor (or ask Claude to do this for you)
2. Open any browser, press F12 to open developer console, and run:
   ```js
   crypto.subtle.digest('SHA-256', new TextEncoder().encode('yournewpassword'))
     .then(buf => console.log(Array.from(new Uint8Array(buf)).map(b=>b.toString(16).padStart(2,'0')).join('')))
   ```
3. Copy the long string it prints out
4. In `index.html`, find this line near the top of the `<script>` section:
   ```js
   const AUTH_HASH = '3072f285c85f73686432ab4e467a48dea7b03e5a02f1042a70ce3919a5e987f8';
   ```
5. Replace the hash with your new one, save, and re-upload to GitHub

The password is never stored in plain text in the file — only its scrambled (hashed) fingerprint, so it can't be read by just viewing the page source.

Once someone enters the correct password, they stay logged in for that browser tab/session — they won't need to re-enter it every time they reopen the same tab, but it will ask again in a fresh tab or after closing the browser.

---

## ☁️ Free cloud data storage (Supabase)

By default, the app stores data **only in memory** — it resets when the page refreshes. To make data persist and sync across every coach's phone/browser, connect a free Supabase database (takes about 5 minutes, no credit card required).

### Step 1 — Create a free Supabase project
1. Go to [supabase.com](https://supabase.com) and sign up (free)
2. Click **New Project** — name it anything, choose any region, set a database password (save it somewhere safe)
3. Wait ~2 minutes for the project to spin up

### Step 2 — Create the data table
1. In your Supabase project, go to the **SQL Editor** (left sidebar)
2. Click **New query**, paste this, and click **Run**:
   ```sql
   create table bvb_data (
     id text primary key,
     payload jsonb,
     updated_at timestamptz default now()
   );

   alter table bvb_data enable row level security;

   create policy "Allow all access"
     on bvb_data
     for all
     using (true)
     with check (true);
   ```
   *(This creates one shared table and allows your app's anonymous key to read/write it — fine for a single-team internal tool behind your password screen.)*

### Step 3 — Get your API keys
1. In Supabase, go to **Project Settings → API**
2. Copy the **Project URL** (looks like `https://xxxxx.supabase.co`)
3. Copy the **anon public** key (a long string starting with `eyJ...`)

### Step 4 — Add the keys to your app
1. Open `index.html`
2. Find these two lines near the top of the `<script>` section:
   ```js
   const SUPABASE_URL = '';
   const SUPABASE_ANON_KEY = '';
   ```
3. Paste your values in between the quotes:
   ```js
   const SUPABASE_URL = 'https://xxxxx.supabase.co';
   const SUPABASE_ANON_KEY = 'eyJhbGciOiJI...';
   ```
4. Save and re-upload `index.html` to GitHub

That's it. From now on:
- Every change (add player, edit benchmark, etc.) auto-saves to your Supabase project a fraction of a second after you make it
- When anyone opens the app and logs in, it pulls the latest data from the cloud automatically
- A small cloud icon next to the club name in the sidebar shows sync status (green check = saved, red = couldn't save — usually means no internet)

**Supabase free tier** comfortably covers a single team's worth of data (500MB database, 2GB file storage, 50,000 monthly active users) — you won't hit limits with normal use.

### Without Supabase (no setup)
If you skip this section entirely, the app still works fully — it just won't remember data between page refreshes or sync across devices. Good for testing things out before committing to the cloud setup.

---

## 📱 Installing on iPhone

1. Open the GitHub Pages URL in **Safari** (must be Safari for iOS install)
2. Tap **Share** → **"Add to Home Screen"** → **Add**

## 📱 Installing on Android

1. Open the URL in **Chrome**
2. Tap **⋮** → **"Add to Home screen"** / **"Install app"**

## 💻 Using on desktop/web

Just open the GitHub Pages URL in any browser. The layout automatically switches between a full sidebar (desktop) and bottom tab bar (mobile).

---

## 🗂 What each file does

| File | Purpose |
|---|---|
| `index.html` | The entire app — UI, styles, logic, auth, and cloud sync |
| `manifest.json` | Tells phones how to install the app (name, icon, colors) |
| `service-worker.js` | Enables offline use by caching app files |
| `icons/` | App icons shown on the home screen |

## 🔄 Updating the app later

1. Edit `index.html` (or ask Claude to help)
2. Upload the new version to the same GitHub repo, overwriting the old file
3. GitHub Pages redeploys automatically within a minute
4. If you bump the `CACHE_NAME` value inside `service-worker.js` (e.g. `v2` → `v3`), phones will be sure to pick up the new version instead of serving a stale cached copy

## 🛟 Troubleshooting

**"Uncaught error" in browser console** — usually means a stale cached version is loaded. Bump the version number in `service-worker.js`'s `CACHE_NAME`, re-upload both files, then on the phone/browser do a hard refresh (or uninstall/reinstall the PWA).

**Cloud sync icon shows red** — check that `SUPABASE_URL` and `SUPABASE_ANON_KEY` are both filled in correctly and that the `bvb_data` table and policy were created exactly as shown above.

**Forgot the password** — open `index.html`, generate a new hash for a password you'll remember (see steps above), replace `AUTH_HASH`, re-upload.
