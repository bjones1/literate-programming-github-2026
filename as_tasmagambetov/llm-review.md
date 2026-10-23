# LLM review

Prompting Claude Opus 5.5 (`claude-opus-5-5`, Claude desktop app) with `Review spec.md: find ambiguous or unspecified behavior, check that every example matches the prose, then implement top_words from the spec alone and run it against every example` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

The same conversation also applied the fixes requested in the PR review, so this is the review of the revised `spec.md`.

---

## Verification

I implemented `top_words` using only the rules in the spec and ran it against every example: the 34 table rows and the two ten-word limit examples.

| Environment    | Unicode | Result        |
|----------------|---------|---------------|
| Python 3.10.12 | 13.0.0  | 36/36 passed  |
| Python 3.13.17 | 15.1.0  | 36/36 passed  |

To check that the new examples rule out other plausible readings, I changed one rule at a time and reran the examples:

| Changed rule                                        | Example that fails                    |
|-----------------------------------------------------|---------------------------------------|
| No second NFC after `lower()`                       | `"ǰ J\u030C"`                         |
| `casefold()` instead of `lower()`                   | `"Straße STRASSE"`, `"ΑΣ'Α"`          |
| `lower()` on the whole text instead of each word    | `"ΑΣ'Α"`                              |
| Only `Mn` and `Me` count as marks                   | `"किताब"`                             |
| `type(text) is str` instead of `isinstance`         | `StrSub("Cat cat")`                   |

## Findings

The prose and the examples agree, and I found no input whose output the spec leaves open.

1. ~~Step 1 (NFC before extraction) is now redundant. Step 3 normalizes every word again, and checking every code point on its own and between two letters found no input where removing step 1 changes the result. Consider dropping it.~~
   *Kept: it is harmless and keeps the processing order easy to follow.*
2. ~~The list under "do not transform words" (no transliteration, no stemming, no stop-word removal, and so on) follows from the sentence before it, and each item has an example. Consider shortening it.~~
   *Kept: stop words and single-letter words are the most likely silent assumptions for a "Trending words" box, so they stay explicit.*
3. ~~"Use `str.lower()`, not `str.casefold()`" repeats step 3.~~
   *Kept for emphasis; the `"Straße STRASSE"` row tests it.*

## Conclusion

The spec is precise enough to implement and test. The remaining comments are optional.
