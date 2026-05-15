# KHANGLAOS v2.2 — Single-File Warehouse System
## ຄັງລາວ · Pro-grade security · EATLAOS-pattern deployment

---

## What this is

A complete restaurant warehouse management system in **one HTML file**, just like EATLAOS. Built on:

- **Frontend:** Single `index.html` (HTML + CSS + JS inline)
- **Backend:** Supabase (PostgreSQL with Row-Level Security)
- **Hosting:** GitHub Pages → `khanglaos.eatlaos.com`
- **Pattern:** Hash routing (`#/login`, `#/dashboard`, `#/receive`, `#/checkout`)

---

## ✅ What works in this version

| Feature | Status |
|---|---|
| Login with JWT (8h expiry, lockout after 5 fails) | ✅ |
| Dashboard (KPIs, FEFO queue, recent movements) | ✅ |
| Receive stock + barcode + printable sticker | ✅ |
| Check-out with phone camera scanner | ✅ |
| FEFO-suggested batch picking | ✅ |
| Waste tracking with reason codes | ✅ |
| Optimistic locking (race-safe checkouts) | ✅ |
| 5 languages (EN/LO/TH/ZH/KO) live toggle | ✅ |
| Mobile + desktop responsive | ✅ |
| Print-ready sticker (70mm thermal or A4) | ✅ |
| Row-Level Security on every table | ✅ |
| Audit log on every write (cannot be bypassed) | ✅ |
| Append-only movements + audit | ✅ |
| Content Security Policy headers | ✅ |
| Zero hardcoded secrets, zero inline event handlers | ✅ |
| **34/34 security audit tests passing** | ✅ |

---

## Project layout (matches EATLAOS exactly)

```
khanglaos/
├── index.html          ← ALL frontend (login + dashboard + receive + checkout)
├── CNAME               ← khanglaos.eatlaos.com
├── README.md           ← this file
├── .gitignore
├── db/
│   └── schema.sql      ← run once in Supabase, then leave alone
├── docs/
│   └── SECURITY.md     ← threat model + verification
└── tests/
    └── security_audit.php  ← local Postgres test runner
```

---

## Setup (one-time, ~25 min)

### Step 1 — Supabase project + schema

1. https://supabase.com → **New project**
   - Name: `khanglaos`
   - Region: **Southeast Asia (Singapore)**
   - Plan: Free
2. Wait ~2 minutes
3. **SQL Editor → New query**
4. Open `db/schema.sql` from this repo, paste, click **Run**
5. Should say "Success. No rows returned."
6. **Table Editor** → confirm 11 tables visible

### Step 2 — Get your keys

1. Supabase dashboard → ⚙️ Settings → **API**
2. Copy two values:
   - **Project URL** (e.g. `https://abcdefg.supabase.co`)
   - **anon public** key (long JWT string)

### Step 3 — Edit `index.html`

Open `index.html` on your PC. Find these two lines near the top of the `<script>` (search for `SUPABASE_URL`):

```js
const SUPABASE_URL      = 'https://YOUR-PROJECT-ID.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR-PUBLIC-ANON-KEY';
```

Replace with your real values. **Save.**

⚠️ **Do NOT use the `service_role` key.** Only the `anon public` key. The service_role key is the master key — never put it in client code.

### Step 4 — Test locally

1. **Double-click `index.html`** to open in your browser
2. Login: `owner` / `khanglaos123`
3. You should land on the dashboard
4. Click **Receive** → fill the form → click "Receive & Print Sticker"
5. You should see a real barcode appear

**If it works locally, GitHub Pages will work.**

If it doesn't work, open browser DevTools (F12) → **Console** tab. Common issues:
- **Red errors mentioning CORS or 401** = Supabase URL/key wrong in index.html
- **Network errors** = no internet, or anon key wrong

### Step 5 — Upload to GitHub

1. Go to your `khanglaos` GitHub repo (private is fine)
2. Click **Add file → Upload files**
3. Drag the **9 items** (or whatever you have):
   - `index.html`
   - `CNAME`
   - `README.md`
   - `.gitignore`
   - `db/` folder
   - `docs/` folder
   - `tests/` folder
4. Commit message: `KHANGLAOS v2.2`
5. Click **Commit changes**

### Step 6 — Enable GitHub Pages

⚠️ **GitHub Pages on private repos requires GitHub Pro ($4/mo). Free option: make repo public.**

**Making it public is SAFE** because:
- Anon key in code is public-safe (RLS blocks reads)
- No passwords in code
- All security is enforced in the database, not in the JS

To make public:
1. **Settings → scroll to Danger Zone → Change visibility → Make public**
2. **Settings → Pages → Source: Deploy from a branch → main → /(root) → Save**
3. Wait 1-2 minutes
4. Refresh — green box "Your site is live at https://USERNAME.github.io/khanglaos/"
5. Click the URL → test login

### Step 7 — Custom subdomain

1. Where `eatlaos.com` DNS is managed → add CNAME record:
   - **Name:** `khanglaos`
   - **Value:** `YOUR-USERNAME.github.io`
   - **TTL:** 300
2. Back in GitHub → **Settings → Pages → Custom domain:** `khanglaos.eatlaos.com` → Save
3. Wait 5-30 minutes for DNS propagation
4. Once green check appears, **enable "Enforce HTTPS"**
5. Done — site live at `https://khanglaos.eatlaos.com`

### Step 8 — Change passwords (DO TODAY)

In Supabase SQL Editor:

```sql
UPDATE users SET password_hash = crypt('your-strong-owner-pw',    gen_salt('bf', 12)) WHERE username='owner';
UPDATE users SET password_hash = crypt('your-strong-sa-pw',       gen_salt('bf', 12)) WHERE username='sa';
UPDATE users SET password_hash = crypt('your-strong-manager-pw',  gen_salt('bf', 12)) WHERE username='manager';
UPDATE users SET password_hash = crypt('your-strong-receiver-pw', gen_salt('bf', 12)) WHERE username='receiver';
UPDATE users SET password_hash = crypt('your-strong-staff-pw',    gen_salt('bf', 12)) WHERE username='staff';
```

---

## Daily use

### Receive a delivery
1. Go to **Receive** tab
2. Pick item (location + expiry auto-fill from defaults)
3. Enter quantity, supplier
4. Click "Receive & Print Sticker"
5. Print sticker → stick on the package

### Check-out for kitchen
1. Go to **Check-Out** tab
2. Click "Start scanner" or type barcode
3. Confirm batch, enter quantity, click confirm
4. Done. Movement logged, batch decremented.

### Throw away expired food
1. Same as check-out, but change "Movement type" to WASTE
2. Pick reason (expired/spoiled/damaged/etc.)
3. Confirm

### Owner morning routine
1. Open dashboard
2. Check 4 KPIs (active batches, expiring soon, expired, low-stock)
3. See FEFO queue — what's expiring soonest
4. See recent movements — what staff did yesterday

---

## Updating the site

1. Edit `index.html` locally on your PC
2. Go to GitHub repo → click `index.html` → pencil icon (Edit)
3. Paste new content → "Commit changes"
4. Wait ~30 seconds → site updated

**One file. One commit. One deploy.** Same as EATLAOS.

---

## Security verification

Before every deploy, run the audit:

```bash
cd khanglaos
php tests/security_audit.php
```

Must show: `RESULTS: 34 passed, 0 failed`

See `docs/SECURITY.md` for full threat model + verification queries.

---

## Phase 2 / future features

Coming in v2.3+:
- Items / Suppliers / Locations management UI (currently you add via Supabase Table Editor)
- Inventory browser with filters
- Reports (waste %, turnover, top movers, supplier scorecards)
- Bulk receive (20 items at once)
- Bulk sticker print (24-per-A4)
- 2FA on SA accounts
- WhatsApp expiry alerts
- Recipe-based depletion linked to EATLAOS POS
- Cycle counts UI
- Mock recall tool
- PWA install + offline mode

---

## License

Private. Built for Bill (boss).

v2.2 · Single-file edition · May 2026 · KHANGLAOS · ຄັງລາວ
