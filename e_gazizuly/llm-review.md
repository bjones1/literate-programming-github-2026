# LLM review of spec.md

Author: e_gazizuly

## Model and process

Reviewer: OpenAI GPT-6-based Codex assistant in this chat. The exact deployed model identifier and reasoning setting are not exposed to the assistant; this record does not claim a more specific model variant. If the course requires that identifier, record the model shown in the chat's model selector before submission.

The same assistant drafted and reviewed the specification; this was not an independent reviewer. The review below concerns the final `spec.md` in this directory.

## Review prompt

> Review the complete accompanying spec.md against the Top words exercise. Look for contradictions, ambiguous inputs or outputs, missing Unicode/tokenization rules, nondeterministic ordering, cutoff behavior, and decisions lacking distinguishing examples. Report remaining issues and their severity. Do not assume behavior that is not written in the specification.

## Latest review output

No blocking ambiguity or contradiction found within the specified runtime contract.

1. **Interface:** The function name, module, argument type, return container, tuple structure, and invalid-input behavior are explicit. The specification accepts string subclasses and excludes implicit conversion or decoding.
2. **Normalization:** The order NFC → casefold → NFC is explicit. Returned words use the resulting spelling. The canonical-equivalence and German case-folding examples distinguish this behavior from raw case-sensitive counting or simple lowercasing.
3. **Tokenization:** Maximal runs of `str.isalpha()` characters define words. Separators split rather than concatenate adjacent letter runs. Examples cover whitespace, punctuation, emoji, apostrophes, hyphens, digits, underscores, and a combining mark that remains after NFC. This intentionally splits contractions and some decomposed letter sequences; that behavior is stated and illustrated.
4. **Languages:** English, Russian, and Kazakh are covered, including mixed-language text. Other scripts are accepted by the same rule. No implicit stemming, transliteration, stop-word removal, or merging of `е` and `ё` occurs.
5. **Ranking:** Counts descend; ties use Python string order. Sorting precedes truncation. Both an all-tied cutoff and a high-frequency word beyond the first ten alphabetical words have distinguishing examples.
6. **Boundary cases:** Empty input, input without letters, fewer than ten words, and invalid values have exact outcomes. Plain text is explicitly distinguished from HTML parsing.
7. **Portability limitation:** Unicode classification follows the running Python version, so newly classified characters can differ across Python releases. The specification explicitly permits this dependency; its acceptance examples use established characters. Pin a Python/Unicode version if identical behavior across every future runtime becomes a requirement.
8. **Example coverage:** The table distinguishes the substantive counting and output decisions. Module naming and absence of external side effects are structural requirements rather than properties demonstrated by output rows.

No review statements have been struck through: none of the above findings were rejected as incorrect or irrelevant.

## Implementation check

Implementation-generation prompt:

> Implement top_words from the complete accompanying spec.md using only the Python standard library. Then evaluate every acceptance-table input and compare its result or exception with the stated expectation. Also check that a str subclass is accepted.

The assistant produced a temporary Python implementation using Unicode normalization, an `isalpha()` scan, `collections.Counter`, and sorting by `(-count, word)`. A verification script parsed the literal inputs and expected results directly from `spec.md` and executed the comparisons.

Observed result: **all 24 table examples passed**, including the three `TypeError` cases. An additional string-subclass check passed. The check ran under Python 3.13. The temporary implementation is a validation aid and is not part of the two-file submission.
