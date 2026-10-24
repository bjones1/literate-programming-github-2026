# LLM Review of `spec.md`

**Model used:** GPT-6 (OpenAI)

**Prompt used:**

> Review `spec.md` for ambiguous, incomplete, contradictory, or technically incorrect requirements. Check Unicode handling, word boundaries, case normalization, sorting and tie-breaking, invalid inputs, and all examples. For each issue, explain why it matters and provide a concrete example. Distinguish actual defects from optional design decisions. Do not rewrite the entire specification. Identify any significant issues that remain and explain whether the specification is ready for implementation.

## Review output

The specification is well structured and mostly unambiguous. No contradictions or incorrect expected outputs were found in the supplied examples.

### Findings

1. **Unicode handling — minor ambiguity.** The specification defines words as consecutive sequences of Unicode letters but does not specify how combining marks or Unicode normalization are handled. For example, the accented letter `é` can be represented as a single code point or as `e` followed by a combining accent. These representations may be processed differently. This is an edge case rather than a contradiction in the core requirements.

2. **Case normalization — minor ambiguity.** The specification requires case-insensitive counting and lowercase output but does not specify whether Python's `lower()` or `casefold()` should be used. These methods can behave differently for some Unicode characters. Explicitly choosing one would make implementation behavior more predictable.

3. **Word boundaries — sufficiently clear.** Consecutive Unicode letters form a word; punctuation, digits, whitespace, underscores, apostrophes, and hyphens separate words. An additional example such as `abc123def`, expected to produce `abc` and `def`, would improve test coverage but is not essential.

4. **Sorting — clear.** Frequency is sorted in descending order, and ties are resolved by Unicode code-point order. This is deterministic, although it may differ from the dictionary order expected by speakers of particular languages. This is a valid design choice, not a defect.

5. **Invalid input — clear.** Non-string inputs must raise `TypeError`. The specification does not require a specific error message, and that omission does not prevent implementation.

6. **Examples — consistent.** All supplied examples match the stated rules. They cover counting, case-insensitive behavior, multilingual text, empty input, punctuation and digits, and frequency ties. Additional tests for apostrophes, hyphens, and more than ten distinct words would improve coverage.

### Conclusion

No significant defects or contradictions were found. The remaining questions concern optional Unicode edge-case clarification and additional test coverage. The specification is ready for implementation under its stated rules, although explicitly defining Unicode normalization and case conversion would make it more precise.