# 🔧 Maintenance — King's Anchorage Broadcast Prep

Your cheat sheet, future-Elijah. You built this; this is just "how do I change it again?"

## The one thing to remember

It's **one file** (`index.html`) hosted on **GitHub Pages** under your `AwsomT` account.
To ship ANY change: **edit → commit → push.** Pages redeploys in ~1 minute. That's the whole
deploy process.

- Project folder: `~/Guittap Synapse X/07 King's Anchorage/church-yt-dock/`
- Live site: https://awsomt.github.io/kings-anchorage-broadcast-prep/
- Repo: `github.com/AwsomT/kings-anchorage-broadcast-prep`

---

## Update a thumbnail

The tool pulls each thumbnail straight from the `thumbnails/` folder in the repo — so updating
one is exactly what you'd hope:

1. Drop the new image into `thumbnails/` using the **same filename** (e.g. overwrite `sunday.jpg`).
2. Commit and push:
   ```bash
   cd ~/"Guittap Synapse X/07 King's Anchorage/church-yt-dock"
   git add thumbnails/ && git commit -m "Update sunday thumbnail" && git push
   ```
3. Done. The next **Set up this week** uses the new image.

Keep them **1280×720**, under **2 MB**. `.jpg` or `.png` both work (the tool re-encodes to
JPEG), so keeping the same filename means **zero code changes**.

| Service | File it uses |
|---|---|
| 9AM + 11AM Sunday | `sunday.jpg` (shared) |
| 6PM Sunday | `evening.jpg` |
| Midweek | `midweek.jpg` |
| Transformation Thursday | `transformation-thursday.jpg` |

---

## First-time setup — connect it to the church's YouTube

Do this once, signed in as the **Google account that owns the King's Anchorage channel.**

1. **console.cloud.google.com** → **Create Project** → name it `Broadcast Prep`.
2. **APIs & Services → Library** → search **YouTube Data API v3** → **Enable.**
3. **OAuth consent screen** → **Internal** (if they have Google Workspace) — otherwise
   **External**, then add the church email under **Test users.**
4. **Credentials → Create Credentials → OAuth client ID → Web application** (name `OBS Dock`):
   - **Authorized JavaScript origins:** `https://awsomt.github.io`  *(no slash, no path)*
   - **Authorized redirect URIs:** `https://awsomt.github.io/kings-anchorage-broadcast-prep/`  *(full path + trailing slash)*
5. Copy the **Client ID**, then in `index.html` replace:
   ```js
   const CLIENT_ID = "PASTE_YOUR_CLIENT_ID_HERE.apps.googleusercontent.com";
   ```
   with it. Commit + push. **This is the last step before the tool works live.**

> The page prints the exact origin + redirect URI it needs to the browser console on load
> (⌥⌘J in Chrome) — copy from there if anything won't match.

---

## Change a service time / name (or add/remove one)

Edit the `SERVICES` array near the top of the `<script>` in `index.html`:
```js
{ key: "s9", label: "9AM Sunday Service", day: 0, hour: 9, min: 0, thumb: "sunday.jpg" },
// day: 0=Sun 1=Mon 2=Tue 3=Wed 4=Thu 5=Fri 6=Sat   hour: 24-hour clock
```
Give any brand-new service a unique `key`. Commit + push.

---

## Go from test to live

In `index.html`, change `const PRIVACY = "private";` to `"public";`. Commit + push.
(Private = safe while testing — no subscriber notifications.)

---

## Other quick switches

- **Stamp the date on thumbnails:** `const DATE_STAMP = true;` (off by default — the church
  supplies finished art).
- **Delete test / junk broadcasts:** in the dock, click **Find & delete ALL test broadcasts.**
- **Test locally before pushing:**
  ```bash
  cd ~/"Guittap Synapse X/07 King's Anchorage/church-yt-dock" && python3 -m http.server 8000
  # open http://localhost:8000/
  # (the test OAuth client needs http://localhost:8000 as an origin + http://localhost:8000/ as a redirect URI)
  ```

---

## In-depth troubleshooting *(only if something breaks)*

**OAuth / sign-in**
- **Two boxes, two formats — the #1 mistake.** JavaScript origins get **no** slash and no path
  (`https://awsomt.github.io`). Redirect URIs get the **full path + trailing slash**
  (`https://awsomt.github.io/kings-anchorage-broadcast-prep/`).
- `redirect_uri_mismatch` → the redirect URI in Google Cloud doesn't **exactly** match what the
  page sends. Open the console (⌥⌘J); it logs the exact value to paste.
- "App is in testing / sign-in blocked" → add the church account under **OAuth consent screen →
  Test users**, or set the consent screen to **Internal** (Workspace only).
- The OAuth Client ID belongs to the **church's** Google account (the one that owns the channel),
  not yours. If you ever rebuild it, redo it on their account.

**Thumbnails**
- `image not found` in the log → the filename in `thumbnails/` doesn't match the `thumb:` value
  in `SERVICES`.
- Thumbnail step fails → image is over 2 MB or not a valid image. Re-export at 1280×720.

**General**
- **`index.html` is the only file that ships.** There is intentionally no `dock.html` — don't
  recreate a second copy or they'll drift out of sync.
- Tokens are **session-only** (implicit OAuth). The operator signs in once per OBS session —
  that's expected, not a bug.
- Quota is a non-issue: ~100 units per broadcast against a 10,000/day free allowance.
