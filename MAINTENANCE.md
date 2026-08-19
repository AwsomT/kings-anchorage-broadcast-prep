# 🔧 Maintenance & Refresher — King's Anchorage Broadcast Prep

*For future-Elijah. If you haven't touched this in months, read this top to bottom and you'll
be back up to speed in 5 minutes.*

---

## What this even is

A one-button web tool that pre-creates the week's YouTube livestreams for King's Anchorage
(9AM / 11AM / 6PM Sunday, Midweek Wed 7PM, Transformation Thursday 7PM) — each auto-named,
dated, and thumbnailed — so the church never renames a stream or uploads a thumbnail by hand.
It runs as a **Custom Browser Dock inside OBS**. See `README.md` for the full install story.

## The 30-second mental model

- It's **one HTML file** (`index.html`). No server, no backend, no build step, no dependencies.
- It's **hosted on GitHub Pages under your account (`AwsomT`)** →
  `https://awsomt.github.io/kings-anchorage-broadcast-prep/`
- The church's OBS just loads that URL. **Nothing is installed on their computer.**
- It talks straight to the **YouTube Data API** from the browser using the church's Google
  **OAuth Client ID** (pasted into `index.html`). No secret is stored — the Client ID is public
  by design.
- **To ship any change: edit `index.html`, commit, push.** GitHub Pages redeploys in ~1 minute.
  That's the entire deploy process.

## Where everything lives

| Thing | Where |
|---|---|
| This project | `~/Guittap Synapse X/King's Anchorage/church-yt-dock/` (inside your Obsidian vault) |
| The one source file | `index.html` |
| Thumbnails | `thumbnails/` — `sunday.jpg`, `evening.jpg`, `midweek.jpg`, `transformation-thursday.jpg` |
| Live site | `https://awsomt.github.io/kings-anchorage-broadcast-prep/` |
| GitHub repo | `github.com/AwsomT/kings-anchorage-broadcast-prep` |

## Common jobs

**Update a thumbnail (church emailed you a new one):**
Drop it into `thumbnails/` with the **exact same filename** (e.g. overwrite `sunday.jpg`) →
commit → push. Done. Keep it **1280×720**, under 2 MB. `.jpg` or `.png` both fine — the tool
re-encodes to JPEG, so the `.jpg` name in config works either way. Same filename = no code change.

**Change a service time / name, or add/remove a service:**
Edit the `SERVICES` array near the top of the `<script>` in `index.html`:
```js
{ key: "s9", label: "9AM Sunday Service", day: 0, hour: 9, min: 0, thumb: "sunday.jpg" },
// day: 0=Sun 1=Mon 2=Tue 3=Wed 4=Thu 5=Fri 6=Sat   hour: 24-hour clock
```
Give any brand-new service a unique `key`. Commit + push.

**Go from test to live:** in `index.html`, change `const PRIVACY = "private";` to `"public"`.
(Private = safe, no subscriber notifications — good while testing.)

**Stamp the date onto thumbnails:** set `const DATE_STAMP = true;`. Off by default because the
church supplies finished art.

**Clean up junk / test broadcasts:** in the dock, click **Find & delete ALL test broadcasts** —
it scans the channel for anything matching the service names and deletes them.

## Testing locally before you push

```bash
cd ~/"Guittap Synapse X/King's Anchorage/church-yt-dock"
python3 -m http.server 8000
# then open http://localhost:8000/
```
For localhost sign-in to work, the OAuth client you're testing with must have
`http://localhost:8000` as a JavaScript origin and `http://localhost:8000/` as a redirect URI.
The page prints the exact origin/redirect it needs to the **browser console** on load
(⌥⌘J in Chrome) — copy from there.

## The OAuth gotchas that will bite you again

- **Two different boxes, two different formats.** JavaScript origins get **no** trailing slash
  and no path (`https://awsomt.github.io`). Redirect URIs get the **full path + trailing slash**
  (`https://awsomt.github.io/kings-anchorage-broadcast-prep/`). Mixing them up is the #1 error.
- `redirect_uri_mismatch` → the redirect URI registered in Google Cloud doesn't **exactly**
  match what the page sends. Open the console; it logs the exact value to paste.
- "App is in testing / sign-in blocked" → add the church account under **OAuth consent screen →
  Test users**, or set the consent screen to **Internal** (Google Workspace only).
- The OAuth Client ID belongs to the **church's** Google account (the one that owns the channel),
  not yours. If you ever rebuild it, redo it on their account.

## Gotchas specific to this tool

- **`index.html` is the only file that ships.** There is intentionally no `dock.html` anymore —
  don't recreate a second copy or they'll drift out of sync.
- Tokens are **session-only** (implicit OAuth). The operator signs in once per OBS session;
  that's expected, not a bug.
- Thumbnails must stay **under 2 MB** after the tool's JPEG re-encode, or `thumbnails.set` fails.
- Quota is a non-issue: ~100 units per broadcast against 10,000/day free.
