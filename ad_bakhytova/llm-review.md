# Final LLM Review — Top Words Specification

## Model used

OpenAI GPT-6 (ChatGPT), final review performed on 2026-10-23.

## Exact prompt used for the final review

Review the accompanying `spec.md` for the Top Words assignment. Identify remaining contradictions, missing edge cases, or behaviors for which two reasonable implementations could return different results. Check whether each decision is supported by an acceptance test. Give a final verdict, and check a Python 3 implementation against every example.

## Complete final review output

**Review scope:** The specification in `spec.md`, including its interface, exact behavior rules, and all 14 acceptance cases.


**Verdict: PASS for the documented behavior.** The specification fixes the major ambiguities in the original exercise: function signature and return type, definition of a word, handling of punctuation/digits/Unicode letters, lowercasing, ties, the ten-result limit, empty text, and invalid input. It requires no third-party dependencies.

**Observations / resolved choices:**

1. "Word" might mean a whitespace-separated token or a string of letters. The spec explicitly chooses maximal runs of Unicode alphabetic characters. Tests for punctuation, hyphens, apostrophes, digits, and underscores make the choice observable.
2. "Most common" leaves ties unclear. The spec fixes descending count then ascending Python Unicode string order; the tie and 11-word tests expose that rule.
3. Case treatment is not given by the original prompt. The spec uses Python `lower()` and has a mixed-case example.
4. Multilingual text could imply language-specific sorting or stop-word filtering. The spec excludes both, with Russian, Kazakh and stop-word examples.
5. Output could be a dictionary, string, or printed text. The spec requires an ordered list of tuples, with full expected list values in the tests.
6. The specification intentionally **does not** normalize Unicode or treat combining marks as part of words. This is explicit, deterministic, and implementable using `str.isalpha()`. For publication-quality multilingual language processing, normalization might be desirable, but adopting it would change this assignment's stated contract.

**Remaining issues:** No unresolved contradictions affecting the stated acceptance examples were found. The choice of letter-only tokenization is intentionally strict, and should not be silently replaced with natural-language tokenization.

## Feedback evaluated and not adopted

- **Use the first appearance to break ties:** Rejected because two implementations could then disagree about expected output; ascending lowercase Unicode string ordering is explicit and independently testable.
- **Strip English stop words:** Not adopted because the exercise asks for frequent words, and language-dependent exclusions would introduce an undefined list and conflict with English/Russian/Kazakh support.
- **Use locale-dependent alphabetical order for Kazakh/Russian:** Not adopted because results could vary between systems. Python string ordering makes tests portable.
- **Apply Unicode normalization and include combining marks within words:** Useful for a real text-processing product, but not adopted in this version because the chosen `str.isalpha()` token rule is precise and the assignment permits a plausible, consistent specification. This is a documented design choice, not a hidden edge case.

## Implementation check

A Python 3 `top_words` reference implementation was produced from the specification and run locally against its examples on 2026-10-23. It scans input characters with `isalpha()`, lowercases each resulting token, counts occurrences, and sorts with `(-count, word)` before taking the first ten.

- 13 expected-output examples: **PASS**.
- 1 exception example (`None` raises `TypeError`): **PASS**.
- **Overall: 14/14 checks passed.**

The reference implementation was used to verify the spec; the assignment submission requires `spec.md` and this final `llm-review.md`. This review records the final specification decisions, rather than claiming that the original task itself prescribed them.
