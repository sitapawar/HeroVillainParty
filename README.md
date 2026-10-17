# HeroVillainParty
# Heroes vs Villains — Live Party Scoreboard

A single-page scoreboard for a Heroes vs Villains Halloween party, made to be projected on a wall. Guests log their deeds through a Google Form. The page reads the linked Google Sheet every 15 seconds and shows team totals, a tug-of-war bar, a live feed of deeds, and a top-players list.

There's no backend, no build step, and no dependencies. It's one `index.html` file that runs on GitHub Pages.

---

## How it works

```
Guest's phone ──► Google Form ──► Google Sheet ──► index.html (projector)
                  (QR code on wall)   (you edit/delete rows here)
```

- **The Google Form** is how guests submit deeds.
- **The Google Sheet** stores every submission, and you manage it directly. Delete a row to remove points, or submit the form yourself to award game points.
- **index.html** reads the Sheet and draws the scoreboard. It never writes anything.

---

## What's on screen

| Area | What it shows |
|---|---|
| **Team totals** | Heroes (bone white) vs Villains (red), animated when points change |
| **Tug-of-war bar** | Each team's share of the total points, with a "Villains lead by N" line |
| **Latest deeds** | The 8 newest submissions: deed, name, witness, time, points |
| **Hall of glory** | The top 5 named players by total points |
| **QR card** | Scan to open the Google Form |
| **Flash** | A large "+N HEROES / VILLAINS" overlay when new points come in |
| **Status** | "Live · Xs ago", or "Reconnecting…" if the Sheet can't be reached |

---

## Form / Sheet columns

The page finds columns by keywords in the header, so small changes to the question wording are fine as long as these words stay in:

| Column | Keyword matched | Used for |
|---|---|---|
| Timestamp | `timestamp` | Feed order and time shown |
| Hero or Villain | `hero or villain` / `team` | Team. Must contain "hero" or "villain" |
| What is your glorious achievement? | `achievement` / `deed` | Feed text |
| How many points was it? | `points` | Points |
| Who is your witness? | `witness` | "witnessed by …" (optional) |
| What's your name? | `name` | Hall of glory (optional) |

### Scoring rules
- Points are rounded and **capped at 10**. Values below 1 or non-numbers are ignored.
- Rows with no recognizable team are ignored.
- A blank name shows as *"A mysterious Hero/Villain"*. Those points count toward the team but not the Hall of glory.
- Names are merged regardless of case or extra spaces ("zoe" = "Zoe").
- A player's team in the Hall of glory is the team on their most recent submission.

---

## Setup

### 1. Share the Sheet
In the Google Sheet: **Share → General access → Anyone with the link → Viewer**.

### 2. Check the config
At the top of the `<script>` in `index.html`:

```js
const SHEET_ID  = '1_IWCVRpsNjjjFByLD9MowAjtP18OBvrA5Ou_2Gzrmz4'; // from the Sheet URL
const SHEET_TAB = 'Form Responses 1';  // tab name at the bottom of the Sheet
const REFRESH_SECONDS = 15;            // how often to check for new deeds
const FEED_LENGTH = 8;                 // deeds shown in the feed
const BOARD_LENGTH = 5;                // players shown in Hall of glory
```

The `SHEET_ID` is the long string in the Sheet URL:
`docs.google.com/spreadsheets/d/`**`SHEET_ID`**`/edit`

### 3. Publish on GitHub Pages
1. Create a public repo and upload `index.html` (and this README).
2. Go to **Settings → Pages → Build and deployment**, choose **Deploy from a branch**, then `main` / `(root)`, and save.
3. After about a minute, the site is live at `https://<username>.github.io/<repo>/`.

You can also just open `index.html` in a browser. It works locally too, but GitHub Pages (HTTPS) is needed for keep-screen-awake.

---

## On the night

| Action | How |
|---|---|
| Full screen | Double-click anywhere, or press **F** |
| Cursor | Hides on its own after 2.5 seconds of no movement |
| Screen sleep | Blocked automatically while the page is open (on HTTPS) |
| Remove bogus points | Delete the row in the Sheet. It disappears on the next refresh |
| Award game points | Submit the form yourself, e.g. *Hero · "Won trivia" · 10 · witness: host* |
| Preview with fake data | Add `?demo` to the URL, e.g. `.../index.html?demo` |

**Before guests arrive:** delete the test rows from the Sheet, then open the page and confirm it shows **0 – 0** and "Live".

---

## Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| Stuck on "Connecting…" / "Reconnecting…" | The Sheet isn't shared as "Anyone with the link", `SHEET_ID` is wrong, or there's no internet |
| Shows 0–0 but the Sheet has rows | `SHEET_TAB` doesn't match the tab name exactly, or the team/points columns were renamed without the keywords |
| A submission isn't counted | Points are missing, not a number, or below 1, or the team is blank |
| Feed shows odd times | The Sheet's locale changed the timestamp format. Points still count |
| QR code goes to the wrong form | The QR is an embedded SVG. Regenerate it for the new form URL and replace the `<svg>` inside `<div class="qr">` |

---

## Technical notes

- **Data loading:** uses the Google Visualization endpoint (`/gviz/tq`) through JSONP, which avoids CORS and returns near-live data. The page falls back to a CSV `fetch` from the same endpoint. This updates faster than "Publish to web", which can be cached for about 5 minutes.
- **Fonts:** Playfair Display and Inter from Google Fonts. Without a connection it falls back to Georgia and the system font.
- **Sizing:** everything is sized in `vh`/`vw`, so the page fills any 16:9 or 16:10 screen without scrolling.
- **Theme:** colors are CSS variables at the top of the `<style>` block (`--bone` for Heroes, `--blood` for Villains, `--ink` for the background).
- **Read-only:** the page never writes to the Sheet. All edits happen in Google Sheets.
