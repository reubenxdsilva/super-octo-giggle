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
  generated: [ [ name, syllables ] ], // names blended from starred seeds
  current:   name | null,             // the card currently on screen
  lastAction:{ type:'yes'|'no', name } // supports one-step Undo
}
```

`seenSet()` = union of `shortlist`, `discarded`, and `removed`. A name in `seenSet` is never suggested again — including a blend, so a passed name never returns by any route.

## The name pool
A hard-coded array `NAMES`, currently ~214 entries, each `["Name", syllableCount]`. Curated to be girl/unisex and easy to pronounce in German. Syllable counts are hand-set (`estSyllables()` is only used for the *last* name, which the user types).

`pool()` = `NAMES` plus `state.generated`, so blends are ordinary candidates once they exist.

## How suggestions are chosen
`pickNext()` scores every unseen candidate that passes the user's filters and shows the highest scorer (with a small random jitter for variety, so the deck isn't identical every run).

`analyzeLastName()` derives, from the current last name: its ending vowel, an estimated syllable count, and its leading consonant cluster ("clash letters", e.g. `Dsilva` → {D, S}).

`scoreName(entry, flow, seeds)` combines:
- **Strong soft filters (−100 each):** the name echoes the surname's ending vowel, or starts with a clash letter. Large enough that these names only appear once everything better is used up — but they are *not* removed from the pool.
- **Rhythm nudge (±):** reward a syllable count different from the surname's, mildly penalize a match.
- **Hard-onset balance (+):** if the surname opens with a consonant cluster, slightly favor vowel-initial first names.
- **Seed nudge (capped at +2):** for each *starred* shortlist name, small bonuses for shared starting letter, syllable count, ending, and similar length.

Only **starred** shortlist names act as seeds (`seedList()`). Un-starred kept names do not influence suggestions. Because `pickNext()` recomputes on demand, the newest stars already affect the next card.

## Blending new names when the pool runs out
When no candidate is left, the Swipe screen shows a **Refresh** button (`refreshFromSeeds()`) instead of a card. It reads whichever names are starred *at that moment* and blends new ones from them. Passed names are never revived.

`blendFromSeeds()` builds candidates two ways, so both halves of every blend come from a real name:
- **onset + rime** — the star's leading consonants plus another pool name's sound from its first vowel on (`Z` + `ophie` → Zophie).
- **opening + tail** — the star's first syllable plus another pool name's final syllable (`Ly` + `ra` → Lyra).

Borrowable parts are filtered so blends stay speakable: a rime carries 1–2 syllables, a tail must start with one plain consonant (or a doubled one, or `ph`/`th`) and end on a vowel or a soft consonant, and `isSpeakable()` rejects consonant/vowel pile-ups and anything shorter than 4 letters. `openingOf()` splits a vowel pair unless it is a diphthong, so `Zoe` opens on `Zo` and `Fay` on `Fay`.

Candidates are ranked with the same `scoreName()` as the pool (so the surname's flow rules and the seed nudges apply), anything with a hard flow violation is dropped while better options exist, and the top `GEN_BATCH` (14) are appended to `state.generated`. `genSyllables()` sets each blend's syllable count, counting a vowel pair once only when it is a diphthong.

## Key functions (all in the `<script>` in index.html)
`load` / `save` (localStorage) · `seenSet` · `analyzeLastName` · `scoreName` · `seedList` · `pool` / `entryFor` / `isGenerated` · `candidates` (filters) · `pickNext` · `onsetOf` / `openingOf` / `tailOf` / `rimeOf` / `isSpeakable` / `genSyllables` / `blendFromSeeds` / `refreshFromSeeds` · `vote` / `undo` / `toggleStar` / `removeFromShortlist` / `resetAll` · `exportShortlist` · render functions per screen · pointer-based drag handling for swiping.

## Known limitations
- Syllable counts for pool names are manual; `estSyllables()` is a simple vowel-group count and can misjudge unusual surnames.
- Blending is recombination, not AI: it can only mix sounds that already exist in the curated pool, and the quality gates are heuristics, so some blends still read oddly.
- Blending needs at least one starred name. With no stars, the empty state has nothing to work from.
- Data and logic live in the same file, which makes the dataset harder to grow (addressed in ROADMAP item 1).
