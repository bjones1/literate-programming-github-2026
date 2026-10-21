# LLM review of spec.md

**LLM used:** Claude Sonnet 5.5 (model id `claude-sonnet-5-5`), used through Claude Code. Round 1 was run in the main session; rounds 2–6 were run by a fresh Claude sub-agent each time (no memory of earlier rounds, only the file). The model serving a sub-agent may differ from the configured one.

**Prompt (every round):** `Review spec.md`

For rounds 2–6 the sub-agent was also told what to look for: errors, contradictions, underspecified behavior, and rules without a discriminating example; to check every example by hand or with Python; and to say explicitly if no real problems remain.

## How many times the spec was checked

The spec was reviewed **6 times**. After each round I fixed the findings (and rewrote the example table), and the examples were run against a reference implementation (`top_words`, about 25 lines of Python written by me from the spec) to make sure every expected output is correct.

| Round | Findings | Main changes made |
|-------|----------|-------------------|
| 1 | 15 | Rule 1 vs rule 2 contradiction; Kazakh tie-break label was wrong (code point order is not Kazakh alphabetical order); added NFC, apostrophe and hyphen variants, digits inside words, underscore |
| 2 | 11 | Stress marks and other combining marks (`ма́ма`) no longer split words; rule 2 written as a grammar; examples written with `\u` escapes; more edge examples |
| 3 | 10 | Sorting on the normalized word (`don’t` vs `dont`); marks not after a letter; NFC, not NFKC (ligature `ﬁ`); final sigma in `lower()` |
| 4 | 8 | `str` subclass wording; soft hyphen; marks around apostrophe and hyphen; NFC mark reordering; complexity claim removed |
| 5 | 7 | Rule 2 prose fixed (it contradicted its own grammar); numeric ideographs are letters; surrogates; `ß`/`ss`; zero-width joiner; return types |
| 6 | 5 | Findings below, all applied |

## Output of the last review (round 6)

Strikethrough would mark output I judged incorrect or irrelevant; I judged none of it incorrect, so nothing is struck. Findings 4 and 5 are minor and were applied as a one-line note and a one-word deletion.

> I found no errors in the rules and no example that contradicts them. I found three real gaps, plus some minor points.
>
> **Example check.** I wrote a reference implementation and ran it with `python3 -I` against examples 1–48 and 50. Examples 49–50 are the `TypeError` and `str`-subclass cases, which I read by hand. Example 3 (Кошка) I verified by hand rather than in the script. Every expected output matched. That includes the tricky ones: 28, 29, 31–34, 38 and 39 (final sigma: `ς` before `-`, `σ` before an inner `'`), 35, 36 and 42. The ordering and cut-off rules (7, 8) are also consistent. I did not edit `spec.md` or read any other file in that directory.
>
> 1. **Rule 4 ("no second normalization") has no discriminating example.**
>    - Example 35 (`İstanbul` → `i̇stanbul`) already comes out in NFC form, so an implementation that wrongly re-applies NFC after `lower()` would still pass.
>    - I confirmed that `"İ̖"` is already NFC (`İ` followed by U+0316). Its `lower()` gives `i̖̇`, which is not NFC, because NFC would put U+0316 before U+0307.
>    - Fix: add an example `"İ̖ i̖̇"` → `[('i̖̇', 1), ('i̖̇', 1)]`, with the decision "two different words; no NFC after `lower()` (rule 4)". Check that row against the reference output; I did not hand-sort it.
>
> 2. **The Python version is unspecified, but the signature needs 3.9 or later.**
>    - The Interface says only "Python 3". `list[tuple[str, int]]` evaluated in a `def` fails on 3.8 and earlier.
>    - Fix: say "Python 3.9+", or allow `from __future__ import annotations`. Also state that the examples assume a Unicode database recent enough for the characters used. Of the examples, only 38 and 39 depend on the `lower()` final-sigma behaviour, and that is stable in all 3.x.
>
> 3. **U+2010 (HYPHEN) is only covered implicitly.**
>    - "Hyphen: only `-`" does exclude U+2010, the real Unicode HYPHEN, but no example or listed exclusion shows it. Implementers often treat it as a hyphen. The listed exclusions are `–`, `—` and U+2011, and examples 11 and 12 use only `–` and U+2011.
>    - I confirmed that `"a‐b"` gives `a` and `b`.
>    - Fix: add U+2010 to the exclusion list in the Definitions, and to example 11, e.g. `"well–known well‑known well‐known"` → `[('known', 3), ('well', 3)]`.
>
> 4. **Minor: rules 3 and 4 can be applied in either order with no visible difference.**
>    - All three apostrophes (U+0027, U+2019, U+02BC) are case-ignorable, so the final-sigma result is the same whichever runs first.
>    - This does not hurt anything, but the stated order has no test behind it. Optionally say "the order of 3 and 4 is not observable" so nobody looks for a hidden requirement.
>
> 5. **Minor: rule 6 names `ª` as a letter with no example.**
>    - Only `三` is exercised, in example 25. `ª` is an Lo character and `isalpha()` is true for it, so it behaves the same.
>    - Fix: optional. Either drop `ª` from the rule text or add it to example 25.
>
> I found no other contradictions or underspecified behaviour. The Interface, the `str`-subclass handling, the mark and apostrophe/hyphen grammar, and ordering and truncation are all consistent with the examples.

## After round 6

All five findings were applied to `spec.md` (new example for `İ` + U+0316, Python 3.9+, U+2010 added, a note on the order of rules 3 and 4, `ª` removed from rule 6). The final `spec.md` was then checked by running all of its examples against the reference implementation (all 51 pass, including the `TypeError` and subclass cases); I did not run a seventh LLM review after these last small edits.
