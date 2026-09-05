# NameSwipe — Architecture

A single-file, offline, vanilla-JS web app for choosing a baby's first name by swiping. Everything lives in `index.html` (HTML + CSS + JS in one file). No build step, no server, no dependencies. State persists in the browser via `localStorage`.

## Run it
Open `index.html` in any modern browser. There is nothing to install.

## The screens
Three tabs via the bottom nav: **Swipe** (one name card at a time, swipe right = keep, left = pass, plus buttons and arrow keys), **Shortlist** (kept names, each star-able and removable, with an Export button), and **Settings** (last name, optional starting-letter filter, optional syllable range, reset).

## Persisted state
One `localStorage` key, `nameswipe.v1`, holding a single JSON object:

```js
{
  settings: { lastName, startLetters, sylMin, sylMax },
  shortlist: [ { name, starred } ],   // names swiped right
  discarded: [ name ],                // names swiped left (never shown again)
  removed:   [ name ],                // taken off the shortlist (never shown again)
  current:   name | null,             // the card currently on screen
  lastAction:{ type:'yes'|'no', name } // supports one-step Undo
}
```

`seenSet()` = union of `shortlist`, `discarded`, and `removed`. A name in `seenSet` is never suggested again.

## The name pool
A hard-coded array `NAMES`, currently ~214 entries, each `["Name", syllableCount]`. Curated to be girl/unisex and easy to pronounce in German. Syllable counts are hand-set (the automatic estimator is only used for the *last* name, which the user types).

## How suggestions are chosen
`pickNext()` scores every unseen candidate that passes the user's filters and shows the highest scorer (with a small random jitter for variety, so the deck isn't identical every run).

`analyzeLastName()` derives, from the current last name: its ending vowel, an estimated syllable count, and its leading consonant cluster ("clash letters", e.g. `Dsilva` → {D, S}).

`scoreName(entry, flow, seeds)` combines:
- **Strong soft filters (−100 each):** the name echoes the surname's ending vowel, or starts with a clash letter. Large enough that these names only appear once everything better is used up — but they are *not* removed from the pool.
- **Rhythm nudge (±):** reward a syllable count different from the surname's, mildly penalize a match.
- **Hard-onset balance (+):** if the surname opens with a consonant cluster, slightly favor vowel-initial first names.
- **Seed nudge (capped at +2):** for each *starred* shortlist name, small bonuses for shared starting letter, syllable count, ending, and similar length.

Only **starred** shortlist names act as seeds (`seedList()`). Un-starred kept names do not influence suggestions. Because `pickNext()` recomputes on demand, the newest stars already affect the next card — but there is currently no explicit "refresh now" control (see ROADMAP).

## Key functions (all in the `<script>` in index.html)
`load` / `save` (localStorage) · `seenSet` · `analyzeLastName` · `scoreName` · `seedList` · `candidates` (filters) · `pickNext` · `vote` / `undo` / `toggleStar` / `removeFromShortlist` / `resetAll` · `exportShortlist` · render functions per screen · pointer-based drag handling for swiping.

## Known limitations
- Syllable counts for names are manual; the last-name estimator is a simple vowel-group count and can misjudge unusual surnames.
- No AI generation — once the well-fitting pool is exhausted for a given surname/filters, you reach an empty state.
- Data and logic live in the same file, which makes the dataset harder to grow (addressed in ROADMAP item 1).
