# alefbet-content

Remote vocabulary content for the [AlefBet](https://github.com/arichazan/HebrewApp) Hebrew-learning app.

The app fetches `v1/vocab_manifest.json` at startup (with an on-device cache and a bundled
fallback baked into the APK, so it always works offline). Editing this file and pushing to
`main` updates the app's content for everyone — **no app release or store update required**.

## Schema

```jsonc
{
  "schemaVersion": 1,     // bump only if the shape of this file changes incompatibly
  "version": 1,           // bump every time you edit content, so clients know to refresh
  "updated": "2026-07-18",
  "courses": [
    {
      "id": "hebrew",              // stable course id the app looks up by
      "letters": [ { "glyph": "א", "name": "Alef", "sound": "...", "mnemonic": "...", "emoji": "🥷", "finalForm": null } ],
      "words": [ { "hebrew": "שָׁלוֹם", "translit": "shalom", "english": "hello / peace", "emoji": "🕊️", "category": "Greetings" } ],
      "expressions": [ /* same shape as words */ ]
    },
    {
      "id": "english",
      "words": [ { "hebrew": "definition text", "translit": "", "english": "target word", "emoji": "💵", "category": "Work & Money", "gloss": "short hint", "example": "example sentence" } ],
      "idioms": [ /* same shape */ ]
    },
    {
      "id": "french",
      "words": [ /* ... */ ],
      "idioms": [ /* ... */ ]
    }
    // Additional course objects (new languages) can be added here. The app's data layer
    // parses every course entry generically; wiring a brand-new course id into the app's
    // navigation/UI is a separate, app-side step (not just a content edit).
  ],
  "categories": [
    { "key": "Greetings", "emoji": "👋", "label": "Greetings" }
    // Add an entry here for any new category key you introduce in a course's items,
    // otherwise the app falls back to a generic ⭐ emoji and the raw category key as the label.
  ]
}
```

## Guidelines

- **Always bump `version`** when you change content — the app uses it to decide whether to
  overwrite its local cache.
- Keep `hebrew`/`translit`/`english`/`emoji`/`category` non-blank for every word.
- Validate the JSON before pushing (`python3 -m json.tool v1/vocab_manifest.json > /dev/null`).
- Breaking the JSON or dropping required fields is safe for users: the app validates the
  fetched manifest and silently falls back to its cached/bundled content if parsing fails.
