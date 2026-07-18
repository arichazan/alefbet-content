# alefbet-content

Remote vocabulary content for the [AlefBet](https://github.com/arichazan/HebrewApp) Hebrew-learning app.

The app fetches `v1/vocab_manifest.json` at startup (with an on-device cache and a bundled
fallback baked into the APK, so it always works offline). Editing this file and pushing to
`main` updates the app's content for everyone — **no app release or store update required**.

This includes adding a **brand-new course/language**: give it valid `courseMeta` fields (see
below) and at least one non-empty `words` list, and it appears in the course picker and gets
working Quiz/Match/Cards/Letters screens automatically — no app code changes needed.

## Schema

```jsonc
{
  "schemaVersion": 1,     // bump only if the shape of this file changes incompatibly
  "version": 1,           // bump every time you edit content, so clients know to refresh
  "updated": "2026-07-18",
  "courses": [
    {
      "id": "hebrew",              // stable course id the app looks up by

      // Course metadata. Required only for courses beyond the app's three built-in ones
      // (hebrew/english/french already have their own hand-tuned screens and ignore this) —
      // but every course below carries it anyway so it's a template to copy for a new one.
      "name": "Hebrew",            // English display name (course picker + home screen)
      "nameEs": "Hebreo",          // Spanish display name (app supports en/es interface locales)
      "tagline": "From your very first alef — letters, words & street slang",
      "taglineEs": "Desde tu primera álef — letras, palabras y jerga callejera",
      "icon": "א",                 // short glyph/text shown as the course "mascot" (not necessarily an emoji)
      "speakLocale": "he-IL",      // BCP-47 tag used for TTS and for the quiz's spoken prompt
      "layout": "direct",          // "direct" (see below) | "reversed"
      "secondaryKind": "expressions", // "expressions" | "idioms" | omit for none

      "letters": [ { "glyph": "א", "name": "Alef", "sound": "...", "mnemonic": "...", "emoji": "🥷", "finalForm": null } ],
      "words": [ { "hebrew": "שָׁלוֹם", "translit": "shalom", "english": "hello / peace", "emoji": "🕊️", "category": "Greetings", "meaningEs": "hola / paz" } ],
      "expressions": [ /* same shape as words */ ]
    },
    {
      "id": "english",
      "name": "English", "nameEs": "Inglés",
      "tagline": "5,600 words and idioms — from everyday to advanced English",
      "taglineEs": "5.600 palabras y modismos — del inglés cotidiano al avanzado",
      "icon": "Aa", "speakLocale": "en-GB", "layout": "reversed", "secondaryKind": "idioms",
      "words": [ { "hebrew": "definition text", "translit": "", "english": "target word", "emoji": "💵", "category": "Work & Money", "gloss": "short hint", "example": "example sentence", "meaningEs": "texto de la definición", "glossEs": "pista corta" } ],
      "idioms": [ /* same shape */ ]
    },
    {
      "id": "french",
      "name": "French", "nameEs": "Francés",
      "tagline": "Already conversational? Enrich your vocabulary — B2-C1",
      "taglineEs": "¿Ya conversas? Enriquece tu vocabulario — B2-C1",
      "icon": "Ç", "speakLocale": "fr-FR", "layout": "reversed", "secondaryKind": "idioms",
      "words": [ /* ... */ ],
      "idioms": [ /* ... */ ]
    }
    // Additional course objects (new languages) go here — see "Adding a new course" below.
  ],
  "categories": [
    { "key": "Greetings", "emoji": "👋", "label": "Greetings" }
    // Add an entry here for any new category key you introduce in a course's items,
    // otherwise the app falls back to a generic ⭐ emoji and the raw category key as the label.
  ]
}
```

### `layout`: `direct` vs `reversed`

- **`direct`** (Hebrew): `hebrew` is the word being learned, spoken in `speakLocale`;
  `english` is its meaning. Quiz prompts with the word, answers are meanings.
- **`reversed`** (English/French-style enrichment courses, for learners who already know the
  basics): `english` actually holds the **target word/idiom**, `hebrew` holds its **definition**
  in plain language, `gloss` is a short hint, `example` is a usage sentence. Quiz prompts with
  the definition, answers are the target words. Use this for any course teaching vocabulary
  *through* definitions rather than direct translation.

### `meaningEs` / `glossEs`: localizing the answer/meaning side

The app's interface language (English/Spanish, from the in-app picker or system default)
should also apply to whichever side of a card/quiz-answer/match-tile is playing the "meaning"
role — the word actually being learned (the `direct`-layout `hebrew` field, or the
`reversed`-layout `english` field) never translates, since that's the point of the exercise.

- `meaningEs`: Spanish version of whichever field is currently the meaning — `english` for
  `direct`-layout courses, `hebrew` (the definition) for `reversed`-layout courses. Used in
  Quiz answers/explanations and Match tiles.
- `glossEs`: Spanish version of `gloss` (the short hint). Only used by Match tiles for
  `reversed`-layout courses, which show the short `gloss` instead of the full definition to
  fit tile space.

Both are optional — a blank or missing value just falls back to the English/French text, so
you can add translations incrementally without breaking anything.

### Adding a new course

1. Pick a stable `id` (lowercase, e.g. `"spanish"`) that isn't `hebrew`/`english`/`french`.
2. Fill in all of `name`, `nameEs`, `tagline`, `taglineEs`, `icon`, `speakLocale`, `layout` —
   a course missing any of these is silently skipped rather than shown broken.
3. Add a `words` list (required — a course with no words never appears) and optionally
   `expressions`/`idioms` (matching `secondaryKind`) and `letters`.
4. Bump `version`, validate the JSON, push.

The course then gets, automatically: a course-picker card, a home screen with XP/streak,
Quiz, Match, and (if `layout: "reversed"`) a flashcards screen — all driven by this file.

## Guidelines

- **Always bump `version`** when you change content — the app uses it to decide whether to
  overwrite its local cache.
- Keep `hebrew`/`translit`/`english`/`emoji`/`category` non-blank for every word.
- Validate the JSON before pushing (`python3 -m json.tool v1/vocab_manifest.json > /dev/null`).
- Breaking the JSON or dropping required fields is safe for users: the app validates the
  fetched manifest and silently falls back to its cached/bundled content if parsing fails.
