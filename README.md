# Emmet Fegan Blitz

A single-page event site for the Emmet Fegan Blitz — 16 clubs, groups,
crests, fixtures, and on-the-day info.

## Files

- `index.html` — the whole site (HTML, CSS, JS, and crest images all in
  one file — the crests are embedded as base64 data, so there's nothing
  else to keep alongside it).
- `assets/badges/` — the original crest image files, kept here for
  reference in case you want to swap one out or re-export.

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

## Running locally

No build tools needed — just open `index.html` in a browser.

## Publishing with GitHub Pages

1. Push this repo to GitHub.
2. In the repo settings, go to **Pages**.
3. Under "Build and deployment", set source to **Deploy from a branch**,
   pick the `main` branch and `/ (root)` folder.
4. Save — the site will be live at
   `https://<your-username>.github.io/emmet_fegan_blitz/` shortly after.
