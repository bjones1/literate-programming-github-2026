# LLM review of `spec.md`

## Model and disclosure

Model: **GPT-6, through Codex**. The same assistant helped draft the
specification, generated a temporary Python implementation, and performed the
reviews below. This is a self-review, not an independent second-model review.
No model version or reasoning-effort setting beyond the available model
identity is claimed.

## Revision before the final review

An earlier verification found that the Kazakh example listed tied words in
an order inconsistent with the specified Unicode code point ordering. The
expected result was corrected to:

```python
[('қазақ', 2), ('үұ', 1), ('һ', 1), ('әң', 1), ('өі', 1)]
```

This correction changes the acceptance example to agree with the ranking
rule. It does not change that rule to Kazakh dictionary order.

## Last review

Prompt used with GPT-6 in Codex, with the complete revised `spec.md` as context:

> Review spec.md against the linked Top words exercise. Check for missing
> requirements, conflicting rules, underspecified behavior, and incorrect
> expected outputs. In particular, check the input and return types, Unicode
> normalization and case folding order, word boundaries, English/Russian/Kazakh
> handling, punctuation, stop words, ties, the ten-word cutoff, and invalid
> inputs. Check that every behavioral decision has a distinguishing example.
> Report actionable defects if any; distinguish optional scope changes from
> defects. Do not write or modify the specification during this review.

Review output:

> I found no remaining actionable defects in the revised specification.
>
> The interface is explicit: a Python 3 function named `top_words` in
> `top_words.py`, taking a string and returning an ordered list of
> `(word, count)` tuples. Non-string inputs raise `TypeError`, and empty or
> letter-free input returns an empty list.
>
> The transformation order is unambiguous: NFC, case folding, NFC, then
> maximal alphabetic runs. The examples distinguish normalization from
> accent removal, case folding from simple lowercasing, and NFC from NFKC.
> Remaining combining marks are explicitly separators; the `İX` example
> shows why transforming before tokenization matters.
>
> The multilingual rules retain distinct Russian and Kazakh letters and
> accept letters from other scripts. Punctuation, digits, apostrophes,
> underscores, hyphens, whitespace, and emoji have explicit boundary
> examples. Mixed scripts and literal markup also have defined outcomes.
>
> Ranking uses descending counts and ascending Python string order, then
> truncates to ten entries. Examples distinguish frequency ordering,
> first-occurrence ordering, prefix ordering, cross-script ordering, and
> ties at the cutoff. The corrected Kazakh example follows this rule.
>
> The table covers the specified behavioral decisions. The absence of
> stop-word filtering and stemming is explicit and illustrated. The
> runtime's Unicode data is identified as the authority for character
> classification, so no locale-specific alphabet is implied.
>
> Linguistic tokenization, a configurable limit, a web interface, and
> performance targets would be additional scope, not necessary fixes.
> Code generation and execution should still be checked against every
> acceptance example; a textual review alone cannot establish that a
> particular implementation is correct.

No parts of this final review were rejected, so no strikethrough is used.
The review does not replace the instructor's feedback or imply PR approval.

## Implementation check

Using the revised specification, the same Codex assistant generated a
temporary Python implementation with NFC normalization, `casefold()`,
`isalpha()` tokenization, counting, and the specified sorting and limit.
A separate checking routine parsed the literal inputs and expected results
from the Markdown table and called the implementation for each row. It
checked exception cases as well as successful return values.

Execution result on Python 3.14.0 with Unicode data 16.0.0:

```text
Checked 34 examples; 0 failures
```

The temporary implementation and checker are kept outside this submission;
the assignment's two submitted files are `spec.md` and `llm-review.md`.
