# NameSwipe

A tiny, offline baby-name chooser. Swipe right to shortlist a first name, left to discard it. It previews each first name against your last name and quietly favors names that flow well with it.

## Run it

Open `index.html` in any modern browser — double-click it, or drag it into a browser window. No install, no server, no build step. All your data stays in the browser (localStorage) on your machine.

## How it works

- **Swipe / buttons / arrow keys** — right = keep, left = pass. Undo brings the last card back.
- **No repeats** — once you've voted on a name it won't show again. Names removed from the shortlist don't come back either.
- **Shortlist** — everything you keep gathers here. Star (★) the ones you love most: only starred names act as *seeds* and nudge which names you see next (shared starting letter, syllable count, endings, length). Nothing is recomputed just because a name was added — only your stars matter.
- **Flow matching** — automatic, derived from the current last name. It downranks (never hard-blocks) first names that echo the last name's ending vowel, match its syllable count too closely, or clash with its opening letters. Change the last name and it re-tunes.
- **Settings** — last name, optional starting letters (letters that clash with the surname are left out by default), and an optional syllable range. That's it; the app handles the rest.

## The name pool

~160 hand-picked girl and unisex first names chosen to be easy to pronounce in German, each tagged with a syllable count. Edit the `NAMES` array near the top of the `<script>` block to add or remove names — format is `["Name", syllableCount]`.

## Notes

- The name shown with the last name is stacked (first name large, surname beneath) so you can read the pairing at a glance.
- There's no AI name generation — it's a fixed curated list so it works fully offline.

## Docs

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — how the app works under the hood
- [`ROADMAP.md`](ROADMAP.md) — planned next features
- [`examples/nameswipe-shortlist.txt`](examples/nameswipe-shortlist.txt) — sample shortlist export
