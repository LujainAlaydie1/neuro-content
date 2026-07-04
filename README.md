# neuro-content

Remote content feed for the [Neuro](https://github.com/LujainAlaydie1) iOS app. The app fetches these files at launch and caches them locally; anything you add or edit here reaches players on their next successful fetch — **no App Store review needed**, since only data changes, not code.

The app always ships with a full bundled copy of this content too, so it works completely offline and even if this repo is ever unreachable — these files are a bonus layer, never a hard dependency.

## Files

### `quiz-content-ar.json` / `quiz-content-en.json`

Arabic and English quiz question banks (kept as **separate files**, not translations of each other — each has its own original questions). Structure:

```json
{
  "questions": [
    {
      "id": "sci-001",
      "category": "science",
      "difficulty": "easy",
      "prompt": "Question text",
      "choices": ["Choice A", "Choice B", "Choice C", "Choice D"],
      "correctIndex": 1,
      "funFact": "One short sentence, a genuine takeaway tied to the correct answer."
    }
  ]
}
```

- `category` must be one of: `science`, `history`, `geography`, `logic`, `popCulture`, `math`, `arabic`, `english`, `space`, `sports`, `technology`, `art`, `nature`, `food`, `music`, `literature`. (`arabic`/`english` here are quiz *topics* — language trivia — unrelated to which JSON file you're editing.)
- `difficulty` must be `easy`, `medium`, or `hard`.
- `correctIndex` is 0-based into `choices`, which must have exactly 4 entries.
- `id` should be unique within the file. Keep it short and prefixed by category (e.g. `sci-`, `his-`) purely by convention — the app doesn't parse the id string.
- To add questions: just append more objects to the `questions` array. To fix a question: edit it in place — the app always fetches the whole file fresh, no versioning needed.
- **Fact-check before merging**, especially anything record-based, date-based, or a superlative claim ("largest", "first", "most") — those go stale. Prefer evergreen facts where possible.

### `daily-quests.json`

The pool of possible daily quests. The app deterministically picks 3 per day from whatever's in this pool (seeded by the date, so it doesn't reshuffle within a day). A bundled default pool of 5-7 ships in the app itself; entries here with a matching `id` **override** the bundled one in place, and any new `id` **extends** the pool.

```json
[
  {
    "id": "quest.finishOneQuiz",
    "titleAr": "أنهِ اختبارًا واحدًا",
    "titleEn": "Finish one quiz",
    "requirement": { "completeActivities": { "count": 1, "type": "quiz" } },
    "bonusXP": 15,
    "bonusCoins": 10
  }
]
```

`requirement` must be one of these three shapes (this mirrors a Swift `Codable` enum exactly — get the shape wrong and the app silently ignores that entry and falls back to bundled defaults, so double-check against these examples):

```json
{ "completeActivities": { "count": 3, "type": "quiz" } }
{ "completeActivities": { "count": 3 } }
{ "logDoomscroll": {} }
{ "reachScore": { "minPercent": 0.8 } }
```

- `type` in `completeActivities` is optional — omit it entirely (don't write `null`) to mean "any activity type." When present it must be one of: `quiz`, `puzzle`, `read`, `memory`, `reflex`, `boss` (only `quiz` is actually playable today; the rest are reserved for future activity types).
- `minPercent` in `reachScore` is a fraction from 0.0 to 1.0.
- Keep `id` stable once published — it's the key the app uses to track a player's daily progress on that quest. Renaming an `id` effectively creates a new quest rather than editing the old one.
- Keep `bonusXP`/`bonusCoins` modest — these are deliberately small, frequent rewards (the existing pool ranges 10–45 XP / 10–25 coins), not a way to inflate the economy.

### `reading-ar.json` / `reading-en.json`

Ultra-short story bank for the Read activity (~100 stories per language, each readable in well under two screen-scrolls). Same "separate, not translated" relationship as the quiz files. Structure:

```json
{
  "stories": [
    {
      "id": "read-en-001",
      "category": "horror",
      "title": "Story title",
      "body": "The full story text, one paragraph."
    }
  ]
}
```

- `category` must be one of: `horror`, `comedy`, `romance`, `marriage`, `family`, `facts`, `scifi`, `fantasy`, `mindset`.
- `id` should be unique within the file; convention is `read-ar-NNN` / `read-en-NNN`, but the app doesn't parse it.
- To add stories: append more objects to the `stories` array. Keep them short — the whole point of this section is a quick, finite reading break, not a long-form article.

### `microgames-content-ar.json` / `microgames-content-en.json`

Remote content banks for the micro-games library (the short "doomscroll-interruption" games, separate from quiz/reading). Only games built around a fixed, swappable list of words/pairs/categories have a section here — games that generate rounds algorithmically (Chaos Calculator, Missing Link, Bubble Pop Math, High-Low Count, Vowel Collector, Grid Density Comparison, Peg Popper, Odd-One-Out Color, Constellation Connect) have nothing to externalize. A key that's missing or empty just means that game keeps using its bundled fallback, so you can add sections one game at a time. Structure:

```json
{
  "rhymeTime": [{ "a": "CAT", "b": "HAT" }],
  "synonymSnap": [{ "a": "HAPPY", "b": "GLAD" }],
  "oppositeAttracts": [{ "a": "HOT", "b": "COLD" }],
  "oddOneOutTypography": [{ "a": "O", "b": "0" }],
  "categorySort": [{ "name": "Animals", "words": ["DOG", "CAT", "LION"] }],
  "wordScramble": [["D", "O", "G"]],
  "colorWordMatch": [{ "colorKey": "red", "label": "RED" }],
  "pairFinderEmoji": ["🐱", "🐶"],
  "emojiEquationEmoji": ["🍎", "🍌"],
  "associationChain": [{ "a": "BREAD", "b": "BUTTER" }],
  "letterNumberSwap": [["C", "A", "T"]]
}
```

- `rhymeTime` / `synonymSnap` / `oppositeAttracts` / `oddOneOutTypography` / `associationChain` are all the same "pair" shape (`a`/`b`), just different games: `a` is what's shown, `b` is the correct answer (for Odd-One-Out Typography, `a` is the repeated base glyph and `b` is the impostor glyph tucked into the grid; for Association Chain, it's an everyday "goes with" pairing like bread/butter, not a synonym or antonym). Keep every `a`/`b` value unique within its own list — the game draws wrong-answer decoys from the same list, so a repeated word can create a round with two "correct" answers.
- `categorySort`: `words` should have at least 4-5 entries per category so decoys don't get thin. Add more categories or more words to existing ones freely.
- `wordScramble`: each entry is one word already split into individual letters, in reading order. Keep words short (3-4 letters) — the game reveals one letter per tap and isn't designed for long words.
- `letterNumberSwap`: same shape as `wordScramble` (a word split into letters), but for a different game — it asks players to sum each letter's numeric value (English: A=1...Z=26; Arabic: traditional Abjad numerals). The app computes the sum itself from whatever letters you provide, so any real word works correctly automatically — no risk of a mismatched hand-authored answer.
- `colorWordMatch`: `colorKey` **must** be one of the app's built-in supported keys — `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `teal`, `pink`, `brown`, `cyan`, `indigo`, `mint` — since that maps to an actual system color in the app; `label` is just the bilingual display text, freely editable. A `colorKey` outside that list is ignored safely (falls back to bundled colors) rather than crashing anything.
- `pairFinderEmoji` / `emojiEquationEmoji`: flat emoji lists, language-neutral (safe to keep identical in both files) — expand freely, native emoji need no art.
- Like the quiz/reading files, editing here **replaces the whole list for that key**, it doesn't merge with the bundled one — so when you add an entry, re-include the ones you want to keep too.
- Seeded at push time with the same content already bundled in the app, so nothing changes for existing players until you actually edit these files.

## Workflow

1. Edit the relevant JSON file directly on GitHub (or clone, edit, `git push`).
2. Validate it's well-formed JSON before pushing (a broken file just means players silently keep using their last-cached-good copy, but it's still worth catching).
3. Players pick up the change automatically the next time their app successfully fetches (currently: once per app launch).
