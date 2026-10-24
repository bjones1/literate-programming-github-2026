# LLM review of spec.md

## Setup

- LLM: Claude Opus 5.5 (`claude-opus-5-5`) in Claude Code (desktop app), default effort.
- Prompt:

  ```
  Review spec.md for the Top words problem. Find errors, contradictions, and
  ambiguities that would let two correct implementations return different
  results. Check every example by hand. For each finding, quote the spec,
  explain the problem, and suggest a fix.
  ```

- Strikethrough marks output that I judged incorrect or irrelevant.

## Output of the last review

I checked all 15 examples by hand against the algorithm. Examples 1–9 and 12–15 are correct. The code point orders that the examples rely on are correct: `с` U+0441 < `қ` U+049B, and `е` U+0435 < `ё` U+0451.

**Should fix**

1. Examples 10 and 11: "the first 10 letters `a`…`j`, each with count 1".
   Problem: These are the only rows whose output is not an exact Python value. A tester cannot compare against them directly, and they are the rows that test the limit of 10.
   Fix: Write the full list literals.

2. Step 1: "then `.lower()`".
   Problem: The spec does not say why it uses `lower()` and not `casefold()`. The two give different results, for example `"Straße"` and `"STRASSE"`. A reader may think that `casefold()` is also allowed.
   Fix: Name `str.lower()` explicitly, say that `casefold()` is not used, and add an example with `ß`.

3. Examples: the Kazakh example uses only one capital letter at the start of a word.
   Problem: Nothing shows that Kazakh-specific capital letters (`Қ`, `Ә`, `Ү`, …) are lowercased.
   Fix: Add a row such as `"ҚАЗАҚ қазақ Қазақ"` → `[('қазақ', 3)]`.

**Minor**

4. "Locale settings must not affect the result."
   Problem: Nothing in the algorithm uses the locale, so the sentence adds nothing.
   Fix: Delete it.

5. ~~Step 1 applies NFC before `lower()` but not after. `lower()` can produce text that is not in NFC, so a second NFC pass is needed.~~

6. ~~"Mark" includes enclosing marks (category `Me`), which are not accents. Exclude `Me` from the definition.~~

7. ~~Add a performance requirement, for example O(n log n), so that slow implementations are rejected.~~

## My notes on the struck-out items

- 5: `lower()` on Latin, Cyrillic, and Kazakh text does not create non-NFC text that would change counts. The rule "NFC, then lower" already defines one exact result, so two correct implementations cannot disagree.
- 6: Enclosing marks almost never occur in English, Russian, or Kazakh text. The rule as written is already exact, so this is a design preference, not a defect.
- 7: The goal of the spec is the result, not the speed. The spec says that any algorithm is acceptable.

I fixed items 1–4 in `spec.md`.
