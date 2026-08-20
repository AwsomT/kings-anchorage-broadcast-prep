# ⚓ King's Anchorage Broadcast Prep

A one-button tool that pre-creates the week's YouTube livestreams for King's Anchorage —
each one **named, dated, and thumbnailed** automatically — so the production team never has
to rename a stream or upload a thumbnail by hand again.

It runs as a panel (a **dock**) inside OBS. Click one button on Sunday morning and all five
services are created on the channel. At service time, just pick the right one in
OBS → **Manage Broadcast** and hit **Start Streaming**.

**Services it creates each week**

| Service | Day | Time |
|---|---|---|
| 9AM Sunday Service | Sunday | 9:00 AM |
| 11AM Sunday Service | Sunday | 11:00 AM |
| 6PM Sunday Service | Sunday | 6:00 PM |
| Midweek Service | Wednesday | 7:00 PM |
| Transformation Thursday | Thursday | 7:00 PM |

---

## Install — add the dock to OBS (one-time)

Do this once, on the church's streaming Mac. Nothing gets downloaded or installed — OBS
just loads a web page.

1. In OBS, go to **Docks → Custom Browser Docks…**
2. **Dock Name:** `Broadcast Prep`
3. **URL:** paste this exact address —
   ```
   https://awsomt.github.io/kings-anchorage-broadcast-prep/
   ```
4. Click **Apply → Close.** The panel appears — drag it anywhere you like in the OBS layout.

That's the whole install.

---

## Weekly use

1. In the **Broadcast Prep** dock, click **Sign in with Google.**
   - Use the account that manages the King's Anchorage YouTube channel.
   - If you see *"Google hasn't verified this app,"* click **Advanced → Continue** — that's
     expected and safe; it's our own in-house tool.
   - You sign in once each time you open OBS.
2. Click **Set up this week.** Each service turns **green** as it's created and thumbnailed.
3. At service time: **OBS → Manage Broadcast → pick the correct service → Start Streaming.**

That's it — no renaming, no thumbnail uploads, ever.

---

## If something looks wrong

| What you see | What to do |
|---|---|
| "Google hasn't verified this app" | Click **Advanced → Continue.** Expected for an in-house tool. |
| Sign-in won't go through / "app is in testing" | The account may need to be added as an approved user — text Elijah. |
| A service shows a **red** dot | Click **Set up this week** again. If it stays red, screenshot it and send to Elijah. |
| Streams are showing as "Private" and you want them public | One-line switch on Elijah's end — ask him to flip it to public. |

---

*Built and maintained by Elijah (AwsomT). Thumbnails, service times, hosting, and the Google
connection are all handled on his end — see **`MAINTENANCE.md`**.*
