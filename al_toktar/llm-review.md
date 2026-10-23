LLM review of `spec.md`
=======================

Model and disclosure
--------------------

Last review model: **DeepSeek V4.1-Flash-1017** 

An earlier review, described below, was performed by \[Claude Sonnet 5.5, e.g. "Claude"\]. Its findings were applied to `spec.md` before the last
review was run.

Revision before the final review
--------------------------------

The earlier review traced the examples against the rules and reported that they
all held, with three problems. Its output was:

> I traced every example against the rules by hand and they all hold, so the fix
> is small. I found three problems:
>
> 1. Stray markup at the end. The document ends with `<p><br></p>`, which
>    doesn't belong in a spec. I removed it.
> 2. `bool` is not covered. `True` is a number type in Python, and the spec
>    lists "numbers" without pinning this down. I added an example row.
> 3. Apostrophe lookalikes are not covered. Only `'` and `’` are treated as
>    apostrophes. `ʼ` (U+02BC) passes `isalpha()`, so it's a letter, while `‘`
>    (U+2018) is a separator. I added a note and an example.

These three changes were applied to `spec.md` before the review below.

Last review
-----------

Prompt used with DeepSeek V4.1-Flash-1017, with the complete revised `spec.md`
as context:

> \["review spec.md", e.g. `Review spec.md`\]

Review output (strikethrough marks output that I deemed incorrect or
irrelevant):

> Overall, `spec.md` is strong: the algorithm is mostly deterministic, the
> ranking rules are clear, and the examples pin down many tricky cases. I did
> not find incorrect expected values in the table. The main issues are a few
> ambiguities and one overbroad non-goal statement.
>
> Findings
> --------
>
> 1. **Apostrophe grammar is slightly ambiguous**
>
>    * "Inside a run, a single apostrophe may join two letters" does not
>      explicitly say whether multiple apostrophes are allowed when each one is
>      individually between letters.
>
>    * E.g. should `"rock'n'roll"` or `"a'b'c"` be one word? The intended answer
>      is almost certainly yes, but the spec should state it formally.
>
>    * Suggested normative wording: after casefold/NFC and U+2019 replacement, a
>      word is a maximal substring matching:
>
>      ```text
>      letter+ (?: ' letter+ )*
>      ```
>
>      where `letter` is any character with `str.isalpha() == True`.
>
>    * This also makes `"a''b"` split into `a` and `b`, and leading/trailing
>      apostrophes separators.
>
>    * Add examples for `"rock'n'roll"` and `"a'b'c"`.
> 2. **"No merging of distinct letters" is too absolute**
>
>    * This conflicts with casefold behavior. `Straße`/`STRASSE` already merge
>      via casefold, and casefold can also map e.g. Kelvin sign `K` to `k`.
>    * More subtly, `İ`.casefold() produces `i` plus a combining dot; under the
>      spec's leftover-combining-mark rule, that combining mark is a separator,
>      so `İ` and `i` can both become `i`.
>    * Suggested rewording: "No additional merging beyond casefold and NFC: no
>      NFKC, no ё→е, no қ→к, no accent stripping, etc."
>    * Consider adding a test for `İ i` if the current behavior is intended.
> 3. **`str` subclass behavior is not fully pinned down**
>
>    * The spec accepts `str` subclasses, but says `text.casefold()`.
>    * A subclass can override `casefold()`, so exact behavior depends on
>      whether the implementation calls `text.casefold()` or
>      `str.casefold(text)`.
>    * Either specify "use `str.casefold(text)` to ignore overrides" or
>      explicitly say overrides are honored.
>    * Add a simple subclass test, e.g. `class S(str): pass;
>      top_words(S("cat")) == [("cat", 1)]`.
> 4. **Missing test for U+02BC**
>
>    * The spec explicitly says `ʼ` U+02BC is a letter, but there is no example.
>
>    * Add something like:
>
>      ```python
>      "don't donʼt" -> [("don't", 1), ("donʼt", 1)]
>      ```
>
>      This pins that U+02BC stays a letter and is not treated as an apostrophe.
> 5. **Minor wording improvements**
>
>    * "A word is a maximal run of letters" is a bit misleading because
>      apostrophes can be inside the word. The formal grammar above would remove
>      that ambiguity.
>    * "leftover combining marks" is fine, but could say "after NFC, any
>      remaining combining mark is a separator."
>    * The type-error rules are clear; no change needed there.
>
> Verdict
> -------
>
> The spec is implementable and the examples are strong. Before freezing it,
> clarify the apostrophe grammar, reword the "no merging" non-goal, and specify
> `str`-subclass/casefold behavior. Add the U+02BC and multiple-apostrophe
> tests.

Implementation check
--------------------

Using the `spec.md` that was reviewed above, Claude Sonnet 5.5 wrote a temporary
Python implementation (NFC after `casefold()`, `isalpha()` tokenization with the
inner-apostrophe rule, counting, then sorting by descending count and ascending
code point, truncated to 10). A separate script parsed every input and expected
result from the Markdown examples table and called the implementation for each
row, including the `TypeError` rows.

Execution result on Python 3.12.3 with Unicode data 15.0.0:

```text
Checked 34 examples; 0 failures
```

The temporary implementation and checker are kept outside this submission; the
assignment's two submitted files are `spec.md` and `llm-review.md`.
