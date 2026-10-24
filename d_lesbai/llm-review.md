# LLM review of spec.md

Prompting Claude Sonnet 5.5 with `Review spec.md` (the final `spec.md` attached) produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

## Result

I checked the spec for contradictions, underspecified behavior and examples that would not distinguish between plausible readings. I found no contradictions and no wrong expected outputs. The remaining remarks are minor:

1. The pipeline order (remove U+00AD and variation selectors → NFC → tokenize → connector normalization → `İ`→`i` → `casefold` → NFC → count → sort) is stated explicitly, and every step whose order matters (`Straße`, `İnternet`, `ΐ`, `co\u00ADoperate`) has an example.
2. The word definition is deterministic, including the edge cases of combining marks (leading mark, mark right after a connector, lone marks), the three apostrophes and the three hyphens. Each rule has at least one example whose output would differ under another plausible reading.
3. Format characters such as the zero-width non-joiner U+200C and the zero-width joiner U+200D (category `Cf`) are separators, so a word containing them is split. This is consistent with the rule "every other character is a separator" but is not mentioned in the examples; it is rare in English, Russian and Kazakh text.
4. "Articles of up to 10 MB are processed in seconds" is a loose bound; the normative requirement is the linear running time, which is clear.
5. The "Known limitations" section lists the removal of U+00AD and variation selectors, which is a rule rather than a limitation; the content is correct, only the placement is odd.
6. A tie between `well` and `well-known` is resolved by code point order (prefix first); this is stated and has an example. Code point order versus language alphabets (Kazakh `ә` after `я`) is also stated and has an example.

## Examples checked against code

A reference implementation written from this spec passes all 46 rows of the examples table (rows are executable test cases in `test_top_words.py`, which reads the table from `spec.md`), plus a linear-time smoke test on a 9 MB input.