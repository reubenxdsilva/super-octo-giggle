# NameSwipe — Roadmap

Two features to build next. Each has a rationale, a design, and acceptance criteria. Paste the relevant section into Cursor as the task. Do them in order — item 1 gives item 2 more to work with.

---

## 1. Bigger, richer name data

**Why.** The pool is ~214 names as bare `["Name", syllables]` pairs baked into `index.html`. That's small and gives the matcher almost nothing to reason about. Move the data out of the code and enrich it, so it can grow to hundreds of names and support smarter filtering and matching.

**Design.**

Move names into a separate `names.json` loaded at startup (keeps data editable without touching logic; easy for both you and an AI to extend). Each entry becomes an object:

```json
{
  "name": "Amelie",
  "syllables": 3,
  "lean": "feminine",        // "feminine" | "unisex"
  "origin": "French",         // rough origin/tradition
  "popularity": 3,            // 1 (rare) … 5 (very common) in the German-speaking region
  "endsWith": "vowel",        // "vowel" | "consonant"  (derivable, but stored for speed)
  "tags": ["elegant", "classic"]
}
```

Keep `syllables` authoritative (do not compute it at runtime for pool names). `endsWith` can be derived on load if you'd rather not store it.

Loading a JSON file over `file://` can be blocked by the browser. To stay true to the "just open index.html" constraint, prefer embedding the data as a `const NAMES = [...]` in a plain `names.js` included with a `<script src>` tag, OR inline a `<script type="application/json" id="names">…</script>` block and parse it. Do **not** switch to `fetch('names.json')` unless you also document running a local server.

Add optional filters in Settings that use the new fields: **origin**, **gender lean** (feminine / include unisex), and a **popularity range** (e.g. "hide very common names"). These are soft where it makes sense and hard where the user clearly means to exclude.

Grow the dataset to 400+ names. Every added name must be girl/unisex and easy to pronounce in German; tag it honestly.

**Acceptance criteria.**
- App still runs by opening `index.html` directly (no server required).
- Name data lives in its own file/block, not inline in the logic.
- Existing users keep their shortlist (localStorage schema unchanged, or versioned + migrated).
- New Settings filters work and combine with the existing starting-letter and syllable filters.
- Dataset ≥ 400 names, each with the full field set.

---

## 2. Refresh suggestions from starred seeds

**Why.** Starred names already nudge the *next* card, but there's no way to say "I've just starred some favorites — re-pick what's coming based on those now." Users want an explicit, visible refresh.

**Design.**

Add a **Refresh** control on the Swipe screen (e.g. a small circular button near the action row, or a pill above the card reading "Seeded by: Zoe, Lynn, Sophie").

On refresh:
- Return the currently displayed card to the unseen pool (it was never voted on, so it must not be marked seen or discarded).
- Re-run selection using the *current* starred seeds and show a fresh top pick.
- Do not touch `shortlist`, `discarded`, `removed`, or settings — refresh only reorders what's upcoming.

Also auto-refresh the upcoming card when the user toggles a star (so starring in the Shortlist tab is felt immediately when they return to Swipe). Show which names are currently seeding suggestions; if none are starred, say so and fall back to normal flow-based ordering.

Optional nicety: build a short look-ahead queue (e.g. next 10) so "refresh" visibly reshuffles it, and so successive cards feel coherent rather than re-rolled from scratch each swipe. If you add a queue, invalidate it on refresh, on star change, and on any settings/last-name change.

**Acceptance criteria.**
- A visible Refresh control on the Swipe screen.
- Refreshing changes the upcoming suggestion(s) to reflect current stars without losing any progress or history.
- Toggling a star updates upcoming suggestions.
- A clear indicator of the active seeds (or a "no seeds yet" state).
- No regression: no-repeat guarantee still holds; the current card isn't lost or double-counted on refresh.

---

## Later (not now)
Offline phonetic scoring (stress/rhyme), learning from full swipe history, AI generation. Keep these out until the two items above are solid.
