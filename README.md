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

## Workflow

1. Edit the relevant JSON file directly on GitHub (or clone, edit, `git push`).
2. Validate it's well-formed JSON before pushing (a broken file just means players silently keep using their last-cached-good copy, but it's still worth catching).
3. Players pick up the change automatically the next time their app successfully fetches (currently: once per app launch).
