# Emmet Fegan Blitz

A single-page event site for the Emmet Fegan Blitz — 16 clubs, groups,
crests, fixtures, and on-the-day info.

## Files

- `index.html` — the page markup and JS. The crest images are embedded
  as base64 data directly in the `teams` array, so there are no separate
  image files to keep track of for those.
- `style.css` — all the site's styling, kept in its own file so
  `index.html` stays readable.
- `assets/badges/` — the original crest image files, kept here for
  reference in case you want to swap one out or re-export.

Note: because the CSS now lives in `style.css`, keep the two files
together when copying or sharing the site — `index.html` alone is no
longer fully self-contained (the crest images still are, since those
stay embedded as base64 data).

## Editing the content

Everything on the page is driven from one `teams` array near the bottom of
`index.html`, inside the `<script>` tag:

```js
const teams = [
  { name: "Belturbet", crest: "assets/badges/Belturbet.png", group: "A" },
  { name: "Colmcille", crest: "assets/badges/Colmcille.png", group: "A" },
  ...
];

const FIXTURE_LIST = [
  { round: 1, time: "10:00–10:15", pitch: "Pitch 1", teamA: "Belturbet", teamB: "Colmcille" },
  ...
];
```

**Groups** are split into Group A and Group B, eight teams each, matching
the official draw. Change the `group` value on a team to move it.

**Fixtures** are the official hardcoded schedule — 6 rounds across 6
pitches, with a couple of pitches sitting idle in rounds 2 and 4 (not
every team plays every round). This isn't a plain round robin, so it's
listed explicitly rather than generated. To change a fixture, edit its
entry in `FIXTURE_LIST` directly — `teamA`/`teamB` reference team names
from the `teams` array above, so crests stay in sync automatically.

**Crests**: each team's `crest` field is a base64 data URI rather than a
file path, so the image is baked directly into `index.html`. To swap a
crest, take the replacement image, base64-encode it (e.g.
`base64 -i newcrest.png` on Mac/Linux, or convert online), and paste the
result into that team's `crest` field as `"data:image/png;base64,...."`
(match the MIME type to the file — `image/png` or `image/jpeg`). This
keeps the page fully self-contained, so it works correctly even if
someone downloads or shares just the single HTML file.

There are also three quick text fields near the top of the page you'll
want to fill in:

```html
<span class="value" id="meta-date">Saturday, TBC</span>
<span class="value" id="meta-venue">Club Grounds, TBC</span>
<span class="value" id="meta-throwin">10:00</span>
```

## Live Fixtures

The "Live Fixtures" section (shown before Groups) lists whichever
matches are currently on the pitch, by checking the visitor's clock
against each fixture's time range in `FIXTURE_LIST`. It shows a red
pulsing beacon when a round is in progress, and a grey beacon with "No
games currently in progress" otherwise. It refreshes every 30 seconds
on its own. From 12:05 (ten minutes after the last round ends at
11:55) it instead shows a "Fixtures finished" message for one hour,
until 13:05 — after that it quietly reverts to the normal idle "No
games currently in progress" state. That window is set by
`FIXTURES_DONE_START` and `FIXTURES_DONE_DURATION_MIN` near
`renderLiveFixtures`.

It's also gated to a single calendar date — the section and its "Live"
nav links stay hidden (`display: none`) on every day except the one set
in `EVENT_DATE` near the top of the `<script>` tag:

```js
const EVENT_DATE = { year: 2026, month: 7, day: 29 }; // month is 0-indexed: 7 = August
```

Update `EVENT_DATE` if the blitz date ever changes.

### Testing it before/after the event

Since the gate and the round times both key off the real clock, you
can't just load the page to see it on any other day. Open the page,
open the browser console (Cmd+Option+J in Chrome, or right-click →
Inspect → Console), and fake the clock:

Paste this **once** per page load to set it up:

```js
const RealDate = Date;
function setFakeNow(y, mo, d, h, mi) {
  Date = function(...args) {
    return args.length ? new RealDate(...args) : new RealDate(y, mo, d, h, mi, 0);
  };
}

// Jump to 10:05 on event day (inside Round 1, 10:00-10:15) and re-render:
setFakeNow(2026, 7, 29, 10, 5);
applyLiveVisibility();
renderLiveFixtures();
```

To change the simulated time after that, only run this (do **not**
paste the block above a second time — see the warning below):

```js
setFakeNow(2026, 7, 29, 11, 20); // Round 5
applyLiveVisibility();
renderLiveFixtures();
```

- Change the date args (`2026, 7, 29`) to a non-event day to confirm
  the section and nav links disappear.
- Pick a time outside any round's window but before 12:05 (e.g.
  `11, 58, 0`) to see the idle "No games currently in progress" state.
- Pick a time between 12:05 and 13:05 (e.g. `12, 30, 0`) to see the
  "Fixtures finished" message.
- Pick a time after 13:05 (e.g. `13, 10, 0`) to confirm it reverts back
  to the idle state.
- Reload the page afterwards to drop the fake clock and go back to
  normal — nothing is saved to the file.

**Warning — don't re-paste the setup block:** Chrome's console shares
`const`/`let` bindings across separate pastes in the same session
rather than treating each paste as its own scope. If you paste the
`const RealDate = Date; function setFakeNow(...) {...}` block a second
time, `RealDate` gets reassigned to the already-overridden `Date`, so
the override ends up calling itself and crashes with `RangeError:
Maximum call stack size exceeded`. If that happens, just reload the
page and paste the setup block fresh — after the first paste, only
ever call `setFakeNow(...)` again to change the time.

On the day itself, no console tricks are needed — it reads the real
system clock automatically.

## Running locally

No build tools needed — just open `index.html` in a browser.

## Publishing with GitHub Pages

1. Push this repo to GitHub.
2. In the repo settings, go to **Pages**.
3. Under "Build and deployment", set source to **Deploy from a branch**,
   pick the `main` branch and `/ (root)` folder.
4. Save — the site will be live at
   `https://<your-username>.github.io/emmet_fegan_blitz/` shortly after.
