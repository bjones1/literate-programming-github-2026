# LLM Review — Top Words Specification

## Model and review context

**Review model:** OpenAI GPT-6 (ChatGPT).

**Review date:** October 24, 2026.

**Source:** The revised `spec.md`, incorporating the instructor's eight comments from an earlier Claude Opus 5.5 (medium effort) review.

## Exact prompt used for the final review

> Review the revised Top Words specification (`spec.md`) as a strict software-specification reviewer. Check whether it responds to all eight instructor comments: (1) Unicode code-point ordering instead of Kazakh alphabetical ordering, (2) interpreting Unicode escape sequences in example inputs, (3) Russian stress-mark and decomposed Cyrillic examples, (4) `str` subclasses, (5) redundant string immutability rule, (6) stopwords, (7) alphabets outside English/Russian/Kazakh, and (8) exact output container types. Check for any remaining contradictions, mismatches between rules and examples, and underspecified behavior. Do not invent requirements; distinguish reasonable design decisions from defects. Check the revised examples against a Python reference implementation. Report the findings and the test outcome.

The `spec.md` submitted alongside this review is the complete specification being reviewed.

## Final LLM review output (GPT-6)

**Overall assessment:** The revised specification consistently defines the input type, word-tokenization process, case folding, counting, tie-break order and output structure. All eight instructor issues have been addressed. The revised examples agree with the stated algorithm in the executed reference implementation. No further blocking contradiction was found in this review.

### Instructor issues verified

1. **Kazakh sorting:** Resolved. Section 6 now specifies comparison by increasing Unicode code point, not by a Kazakh-language alphabet. The explicit `сол сәлем` example returns `[("сол", 1), ("сәлем", 1)]`.
2. **Unicode escape notation:** Resolved. Section 11 identifies its input cells as Python string literals, with surrounding quotes. Therefore the escape in `"café cafe\u0301"` represents a real combining mark rather than literal backslash and digits.
3. **Russian combining marks:** Resolved. Section 11 includes `"моло\u0301ко"` returning `[("ко", 1), ("моло", 1)]` and `"и\u0306"` returning `[("и", 1)]`. The effects are consistent with splitting on non-`isalpha()` characters.
4. **`str` subclasses:** Resolved. Section 9 explicitly requires `isinstance(text, str)`, which accepts a subclass of `str`.
5. **Redundant immutability rule:** Resolved. The bullet requiring the function not to modify its immutable string argument has been removed.
6. **Stopwords:** Resolved. Section 1 states that stopwords remain in the results, even when they dominate counts. This matches the rule in Section 8.
7. **Other scripts:** Resolved. Section 8 explicitly says that characters from other scripts are also tokenized when `str.isalpha()` returns `True`. Support is not restricted to three alphabets.
8. **Concrete output type:** Resolved. Section 2 requires a concrete Python `list` of concrete 2-tuples and exact equality (`==`) with example outputs; generators or custom tuple-like outputs do not qualify.

### Remaining considerations

- Unicode-aware word identification here is deliberately **not** linguistically complete. It can separate decomposed accented or stressed spellings into multiple tokens. This behavior is explicitly documented and illustrated, so it is not a contradiction.
- Unicode code-point tie-breaking is deliberately **not** locale-aware alphabetization. The documentation now says so, preventing accidental implementation changes.
- The specification intentionally provides no strict performance bound or maximum input length. This is explicitly acknowledged as out of scope in the acceptance section.

**Final review decision:** No unresolved issue was identified that prevents implementing the specified behavior deterministically. This is a targeted review, not a formal verification of all possible Unicode strings.

## My evaluation and changes made

I accepted all eight instructor comments and changed the corresponding sections of `spec.md`. I kept the original design choices (no stopword removal, no Unicode normalization, and code-point tie-breaking) because the feedback identified documentation issues, not incorrect requirements. No output from this final LLM review was deemed incorrect or irrelevant, so no strikethrough is used.

## Independent implementation verification

I ran a local Python reference implementation using character-by-character `isalpha()` tokenization, `casefold()` on complete tokens, a `Counter`, and sorting by `(-frequency, word)`.

- **28/28 example rows** in the revised specification matched their expected outputs exactly.
- **3/3 invalid-type checks** (`None`, `123`, `[]`) raised `TypeError` as required.
- A `str` subclass input was additionally accepted, as required.

The verification implementation was used locally and is not included in the PR because the assignment asks for the two Markdown files only.
