# Scout (modified)

Modified Chrome extension build of Scout v0.1.3 → **v0.1.4**.

## What changed

1. **Supabase project swapped.** The bundled Supabase client now points at `https://fzcssialkdybftxmpmhm.supabase.co` instead of the original project. There is now ONE place to edit credentials (see "Required step" below).
2. **Simpler popup UI.** A small CSS override (`assets/scout-simple.css`) flattens the frosted-glass chrome, drops the all-caps display font for body text, and gives buttons/inputs a clean, high-contrast look. **No JavaScript was rewritten**, so every recording / signin / skill-generation flow still works exactly the same.
3. **Version bumped** from `0.1.3` to `0.1.4`.

---

## ⚠️ Required step before it will sign in: paste your anon key

The Supabase **anon public key** is project-specific (it's a JWT with the project ref baked in), so the old key cannot be reused with the new project. The build ships with a placeholder.

Open `assets/supabase-BNjgaoz5.js`. The first 4 lines look like this:

```js
// === Scout Supabase configuration (edit these two lines) ===
const __SCOUT_SUPABASE_URL = "https://fzcssialkdybftxmpmhm.supabase.co";
const __SCOUT_SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImZ6Y3NzaWFsa2R5YmZ0eG1wbWhtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzczMjY3ODcsImV4cCI6MjA5MjkwMjc4N30.DS0UxNbPmBBoMiVeGvQ2S81QOzjsLATq5mA4vFdfpm4";
// ============================================================
```

Replace the `REPLACE_WITH_NEW_PROJECT_ANON_KEY` placeholder with your project's anon public key. You'll find it in your Supabase dashboard:

> **Supabase dashboard → Project Settings → API → Project API keys → `anon` `public`**

It's a long string starting with `eyJhbGciOi...`. Paste it between the quotes. Save the file. Done.

If you ever need to point at a different project later, both the URL and the key are in those same two lines.

---

## Load the extension in Chrome

1. Open `chrome://extensions`.
2. Toggle **Developer mode** on (top right).
3. Click **Load unpacked**.
4. Select the folder containing this README (the one with `manifest.json` in it).
5. Pin the Scout icon to your toolbar.

If you've previously installed Scout v0.1.3, **remove it first** so Chrome doesn't keep two copies side-by-side.

---

## Sign-in note

Sign-in uses the same email + password flow as before — it just authenticates against the new Supabase project instead of the old one. The first time you run the new build:

- If you already have an account on the new project → sign in normally.
- If you don't → create one with the email/password form (same UI as before).

The session is stored in `chrome.storage.local` under a key derived from the project hostname, so the old session won't collide with the new one.

---

## Push to GitHub

From the folder with `manifest.json`:

```bash
git init
git add .
git commit -m "Scout 0.1.4 — point at new Supabase project, simpler popup UI"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

If you'd rather not commit credentials, **don't paste the anon key into the file before committing**. Edit it locally only, or move it to a `.env` you gitignore (would require a small build step — out of scope here).

---

## File map

```
manifest.json                       Manifest V3, version 0.1.4
service-worker-loader.js            Background service worker entry
src/popup/index.html                Popup HTML (now also loads scout-simple.css)
src/offscreen/index.html            Offscreen document for desktop capture
public/icons/*                      Toolbar icons
icons/*                             (Duplicate set — left intact, the manifest references public/icons)
assets/
  ├─ supabase-BNjgaoz5.js          ★ Patched: config block at top
  ├─ scout-simple.css              ★ New: UI simplification overrides
  ├─ index-DGlP9c2M.css            Original Tailwind build
  ├─ index.html-BpL3D_83.js        Popup React bundle (untouched)
  ├─ index.ts-Cy8Vn2AD.js          Service worker bundle (untouched)
  ├─ index.ts-DWOnzrfd.js          Content script bundle (untouched)
  ├─ index.ts-loader-DT7LkkXr.js   Content script loader (untouched)
  ├─ offscreen-COBooiRn.js         Offscreen document bundle (untouched)
  ├─ ids-BvfB8T44.js               UUID helper (untouched)
  └─ modulepreload-polyfill-B5Qt9EMX.js  (untouched)
```

---

## Why I didn't rewrite the popup from scratch

The build is a Vite-bundled output — the React source isn't included, only the minified result. Rewriting the popup as a fresh component would have meant either ripping out the entire IPC contract (`popup:start_recording`, `popup:stop_recording`, status polling, skill rendering, library list, settings, sign-in/sign-out) and reimplementing it, or freezing it as-is. The IPC contract is wired into the service worker, the offscreen document, and the content script — touching it would break recording. So I kept all the logic intact and changed only what's safe to change at the styling layer. You get a calmer, cleaner UI without losing any functionality.
