Prompting GPT-6, High effort, with "Review the following specification: ", produced
the following output; strikethrough indicates LLM output that I deemed incorrect
or irrelevant."
================================================================================

## Specification Review: Top Words

## Overall assessment

**Verdict: Ready for implementation.** The specification is clear, internally consistent, and implementable using Python 3's standard library. It defines tokenization, Unicode normalization, case conversion, validation, frequency counting, sorting, and output formatting precisely.

No blocking contradictions were found between the requirements and examples.

## 1. Requirements analysis

| Requirement | Assessment |
|---|---|
| Function signature | Clear: `top_words(text: str) -> OrderedDict` in `top_words.py` |
| Input validation | Clear: raise `TypeError` for non-`str` inputs |
| Unicode normalization | Clear: NFC |
| Allowed characters | Explicitly enumerated Latin and Cyrillic alphabets |
| Tokenization | Maximal nonempty runs of allowed characters |
| Lowercase conversion | Clear: use `str.lower()` after splitting |
| Mixed-script tokens | Explicitly discarded, not split |
| Frequency storage | A Python `dict` is required |
| Sorting | Deterministic: ascending by `(-frequency, word)` |
| Result size | At most 10 unique valid words |
| Return type | `collections.OrderedDict` |
| Empty input | Covered by an example |

## 2. Important implementation details

### A. Normalize before tokenization

For example:

```python
text = "е\u0308еее"
```

NFC normalization converts this into `"ёеее"`, which is one valid token. If tokenization happens first, the combining mark could incorrectly act as a delimiter.

### B. Discard mixed-script tokens entirely

| Input | Required behavior |
|---|---|
| `helloпривет` | Discard the entire token |
| `hello-привет` | Count `hello` and `привет` |
| `сәлемпривет` | Count as one Cyrillic token |
| `catкот dog` | Count only `dog` |

Latin and Cyrillic letters that look alike must remain distinct.

### C. Lowercase after splitting

The specification requires `str.lower()`, **not** `str.casefold()`. Token validity is checked after lowercasing, but token boundaries are determined using the explicitly allowed characters.

### D. Use the explicit character sets

Do not use `str.isalpha()`, `\w`, or a Unicode-wide letter matcher to determine allowed characters: those would accept letters outside the specified alphabets.

For example, the initial `İ` in `İstanbul` is not allowed and acts as a delimiter, so the remaining `stanbul` is counted.

### E. Use the exact sort key

```python
(-frequency, word)
```

Higher counts sort first. Equal counts are ordered by the lowercase word's Unicode code points using Python's ordinary string comparison.

## 3. Suggested specification improvements

These are clarifications, not defects.

### <s>Improvement 1: State the processing order explicitly</s>
<s>
1. Validate that `text` is a `str`; otherwise raise `TypeError`.
2. Normalize with Unicode NFC.
3. Split into maximal runs of the explicitly allowed characters.
4. Convert each token to lowercase with `str.lower()`.
5. Discard any token containing both Latin and Cyrillic characters.
6. Count valid tokens in a Python `dict`.
7. Sort items by `(-frequency, word)`.
8. Keep the first 10 items.
9. Return them as a `collections.OrderedDict`.
</s>

### <s>Improvement 2: Clarify that NFC does not expand the alphabet</s>

<s>For example, a precomposed accented Latin letter such as `é` is still disallowed and acts as a delimiter. </s>

### <s>Improvement 3: Add tests for wrong input types</s>

```python
top_words(None)  # raises TypeError
top_words(123)   # raises TypeError
top_words([])    # raises TypeError
```

## 4. <s>Additional recommended test cases</s>

| Input | Expected output |
|---|---|
| `"abcАБВ"` | `OrderedDict()` |
| `"abc-АБВ"` | `OrderedDict({"abc": 1, "абв": 1})` |
| `"ҚАЗАҚ қазақ"` | `OrderedDict({"қазақ": 2})` |
| `"a A а А"` | `OrderedDict({"a": 2, "а": 2})` |
| `"hello\nworld\thello"` | `OrderedDict({"hello": 2, "world": 1})` |
| `"12345"` | `OrderedDict()` |
| `"!!!"` | `OrderedDict()` |

<s>
The fourth test is especially useful: Latin `a` and Cyrillic `а` look similar but must be counted separately.
</s>

## 5. Complexity expectations

For an implementation that scans the normalized text, builds a frequency dictionary, and sorts all distinct valid words:

- **Time:** `O(n + k log k)`, where `n` is the text length and `k` is the number of distinct valid words.
- **Space:** Up to `O(n + k)` with ordinary token storage, potentially less temporary storage with incremental tokenization.

No specific performance limit or large-input optimization is mandated.

## Final verdict

**Ready for implementation.** The main pitfalls are incorrect normalization order, splitting rather than rejecting mixed-script tokens, allowing unspecified Unicode letters, and incorrect tie-breaking. Keeping the existing requirements and examples while adding the suggested edge-case tests should be sufficient.
