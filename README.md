# LifeOS

A personal "life OS" built with Flutter — money, tasks, notes, habits and a full Deen
(Quran / dua / hadith / Arabic learning) module, all offline-first on top of Hive.

## Modules

| Area | What it does |
|---|---|
| **Money** | price book with history, debts, shopping list, market parsers |
| **Tasks** | natural-language quick add, scheduling, search |
| **Notes** | rich notes, media, tags, knowledge base |
| **Habits** | streaks, daily routine, timeline |
| **Inbox** | clipboard / screenshot / QR capture with OCR parsing |
| **Deen** | Quran reader (114 surahs, Arabic + Bengali + transliteration), dua, hadith, adhkar, tasbih, prayer times, Arabic alphabet / harakat / roots / grammar, memorization + spaced review |
| **Nova (AI)** | optional assistant for parsing and enrichment |

## Architecture

- Flutter + Dart, Hive for all local storage (~30 boxes opened at startup)
- `lib/screens/…` UI, `lib/services/…` domain logic and stores
- Religious content is **not** bundled: it is downloaded once from a public data repo
  ([`islamic_data`](https://github.com/mdtouajhasanshounto/islamic_data)) via
  `ContentRepository`, validated against its manifest, then cached in Hive

## Run

```bash
flutter pub get
flutter run
```

Optional AI keys (never commit these):

```bash
flutter run --dart-define=LIFEOS_GEMINI_KEY=... --dart-define=LIFEOS_OPENAI_KEY=...
```

## Tests

```bash
flutter test
```

## Status

Work in progress, single-developer project. Not ready for distribution:
no backup/export yet, release signing is still on the debug keystore.
