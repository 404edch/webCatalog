# Catalog Template — Complete Setup Guide

A responsive catalog website for pet adoption, product selling, or any listing.
Built with plain HTML/CSS/JS — no frameworks, no build tools needed.
Hosted on Vercel. Database, images and auth powered by Supabase. All free tiers.

---

## What This Does

- Homepage grid of items with photo, name and price
- Individual detail page per item with photo gallery and specs table
- Contact buttons (email + WhatsApp) on every item page
- Admin panel protected by **hidden login** — no button visible to visitors
- Admin can add/edit/delete items, upload photos, change colors, logo, site name
- All images hosted on Supabase CDN — no localStorage bloat
- Fully responsive on mobile, tablet and desktop
- Auto language detection: Portuguese for PT/BR browsers, English for all others

---

## Stack

| Layer | Service | Cost |
|-------|---------|------|
| Frontend | Single HTML file | Free |
| Hosting | Vercel | Free |
| Database | Supabase Postgres | Free |
| Image storage | Supabase Storage (CDN) | Free |
| Auth | Supabase Auth | Free |

---

## Prerequisites

- A browser and a computer
- Free accounts on: [supabase.com](https://supabase.com), [github.com](https://github.com), [vercel.com](https://vercel.com)
- No credit card needed for any of them

---

## Installation — Step by Step

---

### PHASE 1 — Supabase (database + images + auth)

**Step 1.** Go to [supabase.com](https://supabase.com) → "Start your project" → create a free account.

**Step 2.** Click **"New project"**:
- Name: anything (e.g. `my-catalog`)
- Database password: generate a strong one and save it
- Region: closest to your users (for Brazil → South America / São Paulo)
- Click "Create new project" — wait ~2 minutes

**Step 3.** Create the database tables.

Left sidebar → **SQL Editor** → "New query" → paste the entire block below → click **Run**:

```sql
create table if not exists items (
  id               uuid primary key default gen_random_uuid(),
  name             text not null,
  emoji            text default '📦',
  front_image_url  text default '',
  price            text default '',
  price_required   boolean default false,
  description      text default '',
  specs            jsonb default '[]',
  images           jsonb default '[]',
  created_at       timestamptz default now()
);

create table if not exists site_config (
  id            int primary key default 1,
  site_name     text default 'My Catalog',
  hero_title    text default '',
  hero_subtitle text default 'Browse our listings and get in touch.',
  logo_url      text default '',
  colors        jsonb default '{"primary":"#22c55e","bg":"#fafaf9"}',
  contact       jsonb default '{"email":"","whatsapp":"","address":""}'
);

insert into site_config (id) values (1) on conflict do nothing;

alter table items       enable row level security;
alter table site_config enable row level security;

create policy "public read items"
  on items for select using (true);
create policy "admin write items"
  on items for all
  using (auth.role() = 'authenticated')
  with check (auth.role() = 'authenticated');

create policy "public read config"
  on site_config for select using (true);
create policy "admin write config"
  on site_config for all
  using (auth.role() = 'authenticated')
  with check (auth.role() = 'authenticated');
```

You should see: "Success. No rows returned."

> ⚠️ **If you set up the database in a previous version of this template** (without the `hero_title` column), run this extra line to add it:
> ```sql
> alter table site_config add column if not exists hero_title text default '';
> ```

**Step 4.** Create the image storage bucket.

Left sidebar → **Storage** → "New bucket":
- Name: `catalog-images`
- Toggle **Public**: ON
- Click "Save"

Then create the storage access policies. Go back to **SQL Editor** → New query → paste and run:

```sql
create policy "admin upload"
  on storage.objects for insert
  to authenticated
  with check (bucket_id = 'catalog-images');

create policy "admin delete"
  on storage.objects for delete
  to authenticated
  using (bucket_id = 'catalog-images');

create policy "public read"
  on storage.objects for select
  to anon, authenticated
  using (bucket_id = 'catalog-images');
```

> ⚠️ Always run storage policies via the **SQL Editor**, not via the Storage → Policies visual UI.
> The visual editor does not accept multiple SQL statements and will throw a syntax error.

**Step 5.** Create your admin login.

Left sidebar → **Authentication** → **Users** → **"Add user"** → **"Create new user"**:
- Email: your email address
- Password: choose a password you will remember
- Check **"Auto Confirm User"** ← required, do not skip this
- Click "Create User"

Confirm the user shows as **Confirmed** in the list.

> ⚠️ Do not use **"Invite user"** — the invite email will link to `localhost:3000` and won't work.
> Always use **"Create new user"** with **Auto Confirm User** checked.

**Step 6.** Get your credentials.

Left sidebar → **Settings** → **API**:
- Copy the **Project URL** (looks like `https://abcdefgh.supabase.co`)
- Copy the **Publishable key** (starts with `sb_publishable_...`)

> ⚠️ Do not copy the `service_role` / secret key. That key must never be used in browser code.
> The publishable key is safe for frontend use when RLS policies are enabled (which you just did).

---

### PHASE 2 — Edit index.html

**Step 7.** Open `index.html` in any text editor. Find these two lines near the top of the `<script>` block:

```js
const SUPABASE_URL      = "https://YOUR_PROJECT.supabase.co";
const SUPABASE_ANON_KEY = "YOUR_KEY_HERE";
```

Replace both values with what you copied in Step 6. Save the file.

---

### PHASE 3 — GitHub

**Step 8.** Go to [github.com](https://github.com) → sign in or create a free account.

**Step 9.** Click **+** (top right) → **"New repository"**:
- Name: `my-catalog` (or any name)
- Visibility: **Private** ← this keeps your credentials hidden from the public
- Click "Create repository"

**Step 10.** On the next screen click **"uploading an existing file"** → drag your `index.html` into the area → click **"Commit changes"**.

---

### PHASE 4 — Vercel

**Step 11.** Go to [vercel.com](https://vercel.com) → "Sign Up" → **"Continue with GitHub"** → authorize.

**Step 12.** Click **"Add New"** → **"Project"** → find `my-catalog` → click **"Import"**.

**Step 13.** Configure the deployment:
- Framework Preset: **Other**
- Root Directory: `/`
- Leave everything else as default
- Click **"Deploy"**

Wait ~30 seconds. Vercel gives you a live URL like `my-catalog.vercel.app`.

---

### PHASE 5 — Final configuration

**Step 14.** Copy your live Vercel URL → go to **Supabase** → **Authentication** → **URL Configuration** → paste the URL into **Site URL** → click **Save**.

**Step 15.** Open your live URL in a browser. The catalog should load.

Open it in a private/incognito window to confirm it works for regular visitors (no admin controls visible).

---

## Accessing the Admin Panel

The admin login button is **intentionally hidden** from visitors. Three ways to open it:

| Method | How to use |
|--------|-----------|
| **Keyboard** | Type the word `admin` anywhere on the page (not inside a text field) |
| **URL** | Navigate to `yoursite.vercel.app/#admin` in your browser |
| **Mobile tap zone** | Tap the bottom-right corner of the screen 5 times quickly |

After the login modal appears, enter the email and password you created in Step 5.

Once logged in you will see:
- A green **+** button in the top-right corner to add items
- An **⚙ Admin** button to open the full admin panel
- Edit and delete tools on each item's detail page

---

## Admin Panel — What You Can Change

| Setting | Location |
|---------|---------|
| Site name (shown in nav and footer) | Admin Panel → Contact & Site Text → Nome do Site |
| Hero title (shown large in the homepage) | Admin Panel → Contact & Site Text → Hero Title |
| Site subtitle (shown below hero title) | Admin Panel → Contact & Site Text → Site Subtitle |
| Contact email | Admin Panel → Contact & Site Text → Email |
| WhatsApp number | Admin Panel → Contact & Site Text → WhatsApp (include country code, e.g. `+5541999999999`) |
| Address / location | Admin Panel → Contact & Site Text → Endereço |
| Primary color | Admin Panel → 🎨 Aparência → Cor Principal |
| Background color | Admin Panel → 🎨 Aparência → Fundo |
| Logo image | Admin Panel → 🎨 Aparência → Logo |
| Add new item | Click the green **+** button (top right, admin only) |
| Edit item | Open item page → Admin Tools → ✏ Editar |
| Delete item | Open item page → Admin Tools → 🗑 Excluir |
| Add extra photos to item | Open item page → Admin Tools → + Adicionar Foto |

---

## Updating the Site After Deployment

1. Edit `index.html` on your computer
2. Go to your GitHub repo → click `index.html` → click the pencil icon (Edit file)
3. Select all → delete → paste the new content
4. Click **"Commit changes"**
5. Vercel redeploys automatically in ~30 seconds
6. Hard refresh: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac)

---

## Troubleshooting

### Site loads forever / blank white page

Open browser DevTools (`F12`) → **Console** tab. Find the red error and match it below:

| Error | Cause | Fix |
|-------|-------|-----|
| `Invalid API key` / `401` | Wrong or truncated key | Recopy the full Publishable key from Supabase → Settings → API |
| `ERR_NAME_NOT_RESOLVED` | Wrong Project URL | Recopy the Project URL exactly as shown in Supabase |
| `Failed to fetch` / CORS | Site URL not set | Supabase → Authentication → URL Configuration → add your Vercel URL |
| `forbidden use of secret key` | Used `service_role` key instead of publishable | Use the **Publishable key** only |
| Spins forever, no console errors | SDK version too old | Ensure the CDN line in the file says `@2.50.0` or later |

### Login says "Invalid login credentials"

Possible causes:
- User was created with "Invite user" instead of "Create new user"
- "Auto Confirm User" checkbox was not checked

Fix: Left sidebar → Authentication → Users → delete the user → recreate using **"Create new user"** + check **"Auto Confirm User"**.

### Storage policies fail when using the visual UI

The Storage → Policies visual editor does not accept multiple SQL statements.
Fix: Always run storage policies through the **SQL Editor**, not through the visual policy builder.

### Fields in admin panel don't save (email, site name, hero title, subtitle)

This was a bug in earlier versions of the template where field values would revert to their previous values.
Fix: Make sure you are using the latest version of `index.html` from this repository.

Also required for the Hero Title field to work: run this SQL in Supabase → SQL Editor:
```sql
alter table site_config add column if not exists hero_title text default '';
```

### Images don't upload after logging in

The storage bucket policies were not applied correctly.
Fix: Run the 3 `create policy` statements from Phase 1 Step 4 again in the SQL Editor.

### Site doesn't update after pushing to GitHub

Vercel takes ~30 seconds after a commit. Wait, then hard refresh (`Ctrl+Shift+R`).

### Admin invite email opens localhost

This happens when Site URL is not configured in Supabase before sending the invite.
Fix: Never use "Invite user". Use **"Create new user"** with Auto Confirm instead.

---

## Customizing the Secret Admin Access

Find the `SecretAccess` object in `index.html` (search for `SECRET_WORD`):

```js
const SecretAccess = {
  SECRET_WORD: "admin",   // ← the word to type on keyboard
  SECRET_HASH: "#admin",  // ← the URL hash that triggers login
  TAP_NEEDED:  5,         // ← number of corner taps needed on mobile
  TAP_WINDOW:  3000,      // ← milliseconds to complete the taps
```

Change `SECRET_WORD` to something only you know. Example: `"opensesame"`.

---

## Customizing Colors

The entire color scheme derives from a single primary color. When you change it in the admin panel, all of the following update automatically:
- All buttons and CTA elements
- Hero section background gradient
- Card emoji tile tints
- Empty gallery placeholder gradient
- Input focus rings
- Accent text colors

---

## Adding a New Language

The site detects the browser's default language. To add support for a new language, find the `STRINGS` object (search for `const STRINGS`) and add a block:

```js
const STRINGS = {
  pt: { ... },   // existing Portuguese
  en: { ... },   // existing English
  es: {          // new — copy all keys from "en" and translate
    loading: "Cargando catálogo…",
    admin:   "Admin",
    // ... all other keys
  },
};
```

Then update `detectLang()` (just below `STRINGS`) to recognize the new language:

```js
function detectLang() {
  const nav = (navigator.language || "pt").toLowerCase();
  if (nav.startsWith("en")) return "en";
  if (nav.startsWith("es")) return "es"; // add this line
  return "pt";
}
```

---

## Upgrading Image Storage to Cloudinary (Optional)

If you need more than 1 GB of image storage, you can swap Supabase Storage for Cloudinary (free tier: 25 GB).

1. Sign up at [cloudinary.com](https://cloudinary.com)
2. Go to Settings → Upload → create an **unsigned upload preset**
3. Note your **cloud name** and **preset name**
4. In `index.html`, find the `ImageUpload` object and replace `uploadImage()`:

```js
async uploadImage(file) {
  const form = new FormData();
  form.append("file", file);
  form.append("upload_preset", "YOUR_PRESET_NAME");
  const res = await fetch(
    "https://api.cloudinary.com/v1_1/YOUR_CLOUD_NAME/image/upload",
    { method: "POST", body: form }
  );
  const json = await res.json();
  return json.secure_url;
},
```

No other changes needed — the rest of the code just stores and displays URLs.

---

## Free Tier Limits

| Service | Free limit | Typical usage |
|---------|-----------|---------------|
| Supabase Database | 500 MB | Thousands of items |
| Supabase Storage | 1 GB | ~500 photos at average quality |
| Supabase Auth | 50,000 monthly active users | More than enough |
| Vercel Bandwidth | 100 GB/month | Small catalog uses under 1 GB |

---

## Adding a Custom Domain

1. Vercel dashboard → your project → **Settings** → **Domains**
2. Type your domain and click Add
3. Follow the DNS instructions Vercel shows (usually a CNAME or A record)
4. SSL certificate is issued automatically within a few minutes

---

## Developer Reference — Module Map

The entire application is one file. Use `Ctrl+F` to jump to any section:

| Section | Name | What it does |
|---------|------|-------------|
| CSS 1–9 | Styles | All layout, colors, responsive rules |
| JS 10-A | I18N Module | Language detection + all UI strings (PT/EN) |
| JS 10 | Supabase Config | Your URL and key go here |
| JS 11 | Storage Module | All DB reads/writes — swap here for another backend |
| JS 12 | Image Upload Module | Uploads to Supabase Storage — swap here for Cloudinary |
| JS 13 | State & Render | Single state object + render loop |
| JS 14 | Theme Engine | Derives all colors and gradients from the primary color |
| JS 15–20 | Builder functions | HTML string builders for each page and component |
| JS 21 | Auth Actions | Login/logout via Supabase |
| JS 22 | Item Actions | Add / edit / delete / photo upload |
| JS 23 | Settings Actions | Config, logo, colors — including `saveConfig()` |
| JS 24 | Nav Actions | Page routing (home / detail / admin) |
| JS 25 | Modal Actions | Open/close all modals |
| JS 26 | Helpers | `escHtml`, `escAttr` — XSS prevention |
| JS 27 | Init | Loads SDK, restores session, fetches data, reveals app |
| JS — | SecretAccess | Hidden admin entry: keyboard / URL hash / tap zone |

---

## File Structure

```
my-catalog/
├── index.html    ← the entire application
└── README.md     ← this file
```

No `package.json`. No `node_modules`. No build step. Deploy by uploading one HTML file.
