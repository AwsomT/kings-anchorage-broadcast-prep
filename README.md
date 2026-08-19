# ⚓ King's Anchorage Broadcast Prep

A one-button tool that pre-creates the week's YouTube livestream broadcasts for King's
Anchorage — each one **named, dated, and thumbnailed** automatically — so the production
team never has to rename a stream or upload a thumbnail by hand again.

It runs as a **Custom Browser Dock inside OBS**. Click one button on Sunday morning and all
five services are created on the channel; at service time the operator just picks the right
one in OBS → **Manage Broadcast** and hits **Start Streaming**.

**Services it creates**

| Service | Day | Time | Thumbnail |
|---|---|---|---|
| 9AM Sunday Service | Sunday | 9:00 AM | `sunday.jpg` |
| 11AM Sunday Service | Sunday | 11:00 AM | `sunday.jpg` (shared) |
| 6PM Sunday Service | Sunday | 6:00 PM | `evening.jpg` |
| Midweek Service | Wednesday | 7:00 PM | `midweek.jpg` |
| Transformation Thursday | Thursday | 7:00 PM | `transformation-thursday.jpg` |

---

## How it's deployed

The whole tool is a single self-contained web page (`index.html`) plus a `thumbnails/`
folder. It's hosted for free on **GitHub Pages under Elijah's GitHub account (`AwsomT`)**,
which gives it a permanent `https://awsomt.github.io/...` URL. Nothing runs on the church's
computer — OBS just loads that URL. Reboots, updates, and volunteers changing machines don't
break anything.

- **Maintainer (Elijah, GitHub `AwsomT`):** owns the GitHub repo + Pages hosting. Pushes fixes
  and thumbnail updates. See **Part A** and **`MAINTENANCE.md`**.
- **Church Google account:** owns the YouTube channel and the Google Cloud OAuth client that
  lets the tool act on the channel. See **Part B**.
- **OBS operator:** adds the dock URL once, then uses one button weekly. See **Part C** and
  **Weekly use**.

---

## Part A — Host it on GitHub Pages (one-time, maintainer)

1. Create a **public** GitHub repo (e.g. `kings-anchorage-broadcast-prep`).
2. Push these files to it (`index.html`, `thumbnails/`, this `README.md`).
3. On GitHub: **Settings → Pages → Build and deployment → Source: Deploy from a branch →
   Branch: `main` / `/ (root)` → Save.**
4. Wait ~1 minute. GitHub shows the live URL, e.g.
   `https://awsomt.github.io/kings-anchorage-broadcast-prep/`
   **This URL is used in Part B and Part C — keep it handy.**

> The page needs the OAuth **Client ID** from Part B pasted into `index.html` before it works.
> Order of operations: create the repo/Pages URL first (Part A), register that URL and get the
> Client ID (Part B), then commit the Client ID (end of Part B).

---

## Part B — Google Cloud OAuth (one-time, on the CHURCH's Google account)

This authorizes the tool to create broadcasts on the church's YouTube channel. **Sign in with
the Google account that manages the King's Anchorage YouTube channel.**

1. Go to **console.cloud.google.com** → sign in as the church account.
2. **Create Project** → name it `Broadcast Prep` → select it.
3. **APIs & Services → Library** → search **"YouTube Data API v3"** → **Enable**.
4. **APIs & Services → OAuth consent screen:**
   - If the church has **Google Workspace**: choose **Internal** (simplest — no test-user step).
   - Otherwise choose **External**, fill in App name + a support email, save through the screens,
     then under **Test users → Add users** add the church account email.
5. **Credentials → Create Credentials → OAuth client ID:**
   - Application type: **Web application**
   - Name: `OBS Dock`
   - **Authorized JavaScript origins** → Add (⚠️ **no** trailing slash, no path):
     ```
     https://awsomt.github.io
     ```
   - **Authorized redirect URIs** → Add (⚠️ **with** the full path **and** trailing slash):
     ```
     https://awsomt.github.io/kings-anchorage-broadcast-prep/
     ```
   - **Create** → copy the **Client ID** (ends in `.apps.googleusercontent.com`).

> **Tip:** open the hosted page and press **⌥⌘J** (Chrome) to open the console — it prints the
> exact `Authorized JavaScript origin` and `Authorized redirect URI` to paste. They must match
> **character-for-character**, or you'll get `redirect_uri_mismatch`.

6. Open `index.html`, find this line near the top of the `<script>`:
   ```js
   const CLIENT_ID = "PASTE_YOUR_CLIENT_ID_HERE.apps.googleusercontent.com";
   ```
   Replace it with the Client ID from step 5. Commit + push. GitHub Pages redeploys in ~1 min.

---

## Part C — Add the dock to OBS (one-time, on the church Mac)

1. In OBS: **Docks → Custom Browser Docks…**
2. **Dock Name:** `Broadcast Prep`
   **URL:** your GitHub Pages URL from Part A.
3. **Apply → Close.** The dock panel appears; drag it wherever you like in the OBS layout.

---

## Weekly use

1. In the **Broadcast Prep** dock, click **Sign in with Google** (first time each session).
   - If you see *"Google hasn't verified this app,"* click **Advanced → Continue** — expected
     for an internal tool.
2. Click **Set up this week**. Each service turns green as it's created + thumbnailed.
3. At service time: **OBS → Manage Broadcast → select the correct service → Start Streaming.**

That's it. No renaming, no thumbnail uploads.

> **Privacy note:** while testing, the tool creates broadcasts as **Private** (see
> `PRIVACY` in `index.html`). When you're confident, change `const PRIVACY = "private";` to
> `"public"` and push.

---

> **Maintaining this later?** See **`MAINTENANCE.md`** — a refresher for when you haven't
> touched this in months (where everything lives, how to update, and the OAuth quirks).

---

## How it works (for the curious)

- Pure front-end. No server, no backend, no database.
- Google **implicit OAuth** flow → an access token in the browser (no client secret needed;
  the Client ID is public by design). Tokens are session-only, so sign in each session.
- **YouTube Data API v3**: `liveBroadcasts.insert` sets title + scheduled time;
  `thumbnails.set` uploads the image (drawn to a 1280×720 canvas, exported as JPEG).
- Quota cost is tiny — ~100 units per broadcast against a 10,000/day free allowance.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `redirect_uri_mismatch` | The redirect URI in Google Cloud must exactly match the console-printed value (path + trailing slash). |
| `Invalid Origin: … must not contain a path or end with "/"` | You put the slash version in the **JavaScript origins** box. Origins get **no** slash; redirect URIs get the slash. |
| Sign-in blocked, "app is in testing" | Add the church account under **OAuth consent screen → Test users**, or set the consent screen to **Internal** (Workspace only). |
| `image not found` in the log | The thumbnail filename in `thumbnails/` doesn't match the `thumb:` value in `SERVICES`. |
| Thumbnail step fails | Image over 2 MB, or not a valid image. Re-export at 1280×720. |
