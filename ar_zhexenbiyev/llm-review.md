# LLM review of `spec.md`

## Prompt used

> Review the following specification for the Top words problem. Identify every
> remaining ambiguity, omission, or contradiction that could allow two
> reasonable Python implementations to produce different results. Pay special
> attention to input validation, Unicode normalization and case folding,
> definition of a word in English/Russian/Kazakh, apostrophes, punctuation,
> digits, tie ordering, the ten-result limit, empty input, output shape, and
> whether every important decision has an example with an expected output.
> Review the complete contents of `spec.md` and report concrete fixes, not
> general encouragement.

## Specific LLM used

This review was performed by the AI assistant using Copilot SDK in VS Code.
The SDK session did not expose a model identifier, so no narrower model name
is claimed here.

## Review output

The specification is substantially deterministic and covers the important
choices: type errors, empty input, NFC normalization, Unicode letters,
apostrophes, separators, case folding, counting, tie-breaking, result limits,
and output shape. The examples exercise English, Russian, Kazakh, combining
marks, apostrophes, punctuation, ties, and fewer-than-ten results.

One contradiction was found in the ten-word example: `w1` through `w11` are
not words under the stated rule because digits are separators, and the example
therefore cannot demonstrate the ten-result limit. The specification should
replace it with a letter-only example.

One presentation issue was found in the following boundary example: it
included an extra trailing comma and then described a corrected result in
prose. That creates an unnecessary second interpretation. The example should
show only the formal expected result.

## Resolution

Both issues were resolved in `spec.md`: the invalid alphanumeric example is
explicitly identified as a non-word case, and the letter-only ten-plus-one
example is followed by the exact ten-item result. The contract itself remains
unambiguous.

## Follow-up review

> Review the revised `spec.md` again. Look for contradictions between prose
> and examples, especially normalization of apostrophe variants, representation
> of Unicode examples, the definition of separators, ranking ties, and the
> output contract. Report only concrete remaining gaps.

The revised specification resolves the previous issues. It now explicitly
normalizes all three accepted apostrophe forms to ASCII before case folding,
and the examples contain the actual Unicode characters rather than
backslash-escape text. The apostrophe example therefore has the stated single
`o'reilly` result. No further concrete ambiguity was found in the reviewed
areas.

## Latest review

### Prompt used

> Re-review the complete revised `spec.md` for the Top words assignment. Find
> any remaining behavior that two competent Python implementations could
> choose differently. Check normalization order, Unicode `isalpha` and
> `casefold` behavior, apostrophe handling, invalid inputs, empty results,
> ranking, tie-breaking, the ten-item limit, and whether examples cover every
> interface decision. Report concrete gaps and corrections only.

### Specific LLM used

GPT-5.6 Luna, an AI assistant using Copilot SDK in VS Code.

### Findings and resolution

Two gaps remained. First, the specification did not explicitly say whether
normalization happens after case folding; this matters because case folding can
produce a canonically decomposed result. Second, it referred to Unicode
letters and case folding without explicitly requiring Python's standard
library semantics. The specification now requires NFC after case folding and
requires the standard-library `str.isalpha()`, `str.casefold()`, and NFC
operations. It also adds examples for empty input, separator-only input, and
the `TypeError` contract.

No further concrete ambiguity was found.
