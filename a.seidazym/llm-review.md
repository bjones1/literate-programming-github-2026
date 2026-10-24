# Final LLM review — Top Words specification

**Model:** OpenAI GPT-6 (ChatGPT), October 23, 2026.

**Prompt used:**

> Review the revised `spec.md` for the Top Words assignment after the professor's Claude Opus 5.5 review. Check every issue raised: apostrophes and hyphens in English/Russian/Kazakh; stop words; Python/Unicode version dependence; `str` subclasses; Turkish dotted I; Russian ё versus е; Greek sigma and German ß; orphan and repeated combining marks; wording on side effects and normalization order; completeness of acceptance criteria; input size and complexity; and Kazakh scripts. Identify any remaining ambiguity or incorrect example. Review the exact expected outputs against the written rules.

## Review output

### Findings addressed

1. **Internal punctuation:** ASCII apostrophe, U+2019 apostrophe, and ASCII hyphen are included only when immediately between alphabetic characters. This keeps `it's`, `what's`, `что-то`, and `кое-как` intact. The two apostrophe types remain distinct keys; this is deliberate and is tested.
2. **Stop words:** No filtering is performed; the purpose explicitly describes frequency rather than keyword extraction and acknowledges that function words may dominate.
3. **Unicode version:** Reference behavior is pinned to CPython 3.14.6 / Unicode 16.0. Different Unicode database versions may disagree for some inputs.
4. **`str` subclasses:** Accepted using `isinstance(text, str)`.
5. **Language-dependent normalization:** Turkish `İ`, Russian `ё` versus `е`, Greek sigma, and German `ß` behavior is explicitly described. Output shows normalized keys rather than original spellings.
6. **Combining marks:** Orphan marks are separators; combining marks following a word are included. Examples cover both isolated and repeated marks.
7. **Wording:** The redundant instruction not to modify immutable strings was replaced with a side-effect requirement. The reason for identifying word boundaries before case folding is included.
8. **Acceptance:** Examples illustrate the rules but do not supersede them. The rules remain the source of truth.
9. **Performance:** The entire input is processed; an expected asymptotic performance target is stated, with no arbitrary input cutoff.
10. **Kazakh scripts:** Cyrillic and Latin Unicode input is accepted, but historical apostrophe-based orthographic equivalence is not reconstructed.

### Remaining limitations / qualifications

- This is intentionally a character-based tokenizer, not a full morphological or linguistic word segmenter. Unicode and punctuation policies are product choices rather than universally correct linguistic rules.
- An internal connector is recognized only if its immediate neighbors are alphabetic. Consequently, combining marks between a letter and a connector prevent the connector from joining words; the specification explicitly defines this behavior.
- The Unicode reference environment should be used for authoritative cross-version verification. Tests executed during this review used Python 3.13.5 / Unicode 15.1.0, **not** the pinned reference version. Therefore passing tests do not prove identical results for every character on the reference runtime.

### Verification

An independently written reference implementation was used to run the **25 executable examples** in §5; all 25 passed on Python 3.13.5 (Unicode 15.1.0). The implementation was used for checking only and is not part of the required submission. Additional invalid-type and subclass checks are specified as requirements but were not part of those 25 assertions.

### Conclusion

No contradictions were found among the revised examples and the specified tokenization, normalization, counting, and ordering rules. The earlier review's substantive concerns are explicitly resolved or acknowledged as intentional limitations. The specification is ready for the instructor to review, with the stated cross-version testing qualification.
