# Top Words — Specification

## Purpose

A news website needs a "Trending words" panel. Implement a Python 3 function named `top_words` in a file named `top_words.py`. It returns the ten most frequent words in a single article, or fewer if the article contains fewer than ten distinct words. The input may contain English, Russian, or Kazakh text.

## Interface

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

- `text` must be a Python `str`. For any other type, raise `TypeError`.
- Return a new list of `(word, count)` tuples. `word` is a lowercase Unicode string and `count` is a positive integer.
- Do not print anything, modify the input, read files, or use external packages.

## Exact rules

1. **Word boundaries:** A word is a maximal consecutive sequence of Unicode alphabetic characters (`str.isalpha()` is true for each character). Any other character — including spaces, punctuation, apostrophes, hyphens, digits, underscores, and emojis — separates words. Combining marks are separators under this rule; no Unicode normalization is performed.
2. **Case:** Convert each extracted word using Python's `str.lower()` before counting. Words that become identical after this operation are counted together. Do not stem words, remove stop words, or transliterate.
3. **Frequency:** Count each occurrence, including repeated words.
4. **Ordering:** Sort by descending count. For equal counts, sort by ascending Python string order (`str` lexicographic Unicode code point order) of the lowercased word. Do not depend on first appearance or locale-dependent collation.
5. **Limit:** Return the first 10 entries after sorting. If there are fewer than 10 distinct words, return all of them. For no words, return `[]`.
6. **Determinism:** The same input must always yield the same ordered list. No mutation or persistent state is allowed.

## Acceptance tests

Each expected value is the **entire returned list**, not printed output. Tests intentionally distinguish plausible interpretations of the original short assignment.

| Case and decision | Input (`text`) | Expected result |
| --- | --- | --- |
| Repeated words and counts | `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| Case-insensitive matching | `"Cat CAT cat Dog"` | `[("cat", 3), ("dog", 1)]` |
| Equal counts break alphabetically, not by appearance | `"zebra apple zebra apple"` | `[("apple", 2), ("zebra", 2)]` |
| Punctuation separates words | `"hello,world! hello."` | `[("hello", 2), ("world", 1)]` |
| Apostrophes and hyphens split words | `"don't re-enter"` | `[("don", 1), ("enter", 1), ("re", 1), ("t", 1)]` |
| Digits and underscores separate words; digit-only input is not a word | `"abc123def foo_bar 456"` | `[("abc", 1), ("bar", 1), ("def", 1), ("foo", 1)]` |
| Russian text | `"Привет, мир! ПРИВЕТ"` | `[("привет", 2), ("мир", 1)]` |
| Kazakh text (Cyrillic) | `"Қазақ тілі қазақ ТІЛІ"` | `[("тілі", 2), ("қазақ", 2)]` |
| Empty text | `""` | `[]` |
| No alphabetic characters | `"123 !? _ 😊"` | `[]` |
| Limit to 10; alphabetical tie-break | `"k j i h g f e d c b a"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` |
| More than one whitespace character | `"cat\n\tcat   dog"` | `[("cat", 2), ("dog", 1)]` |
| No stemming or stop-word removal | `"the cats cat the"` | `[("the", 2), ("cat", 1), ("cats", 1)]` |
| Non-string input | `None` | Raises `TypeError` |

## Completion criterion

An implementation is compliant only if it matches every expected result above, including tuple/list types and order, and raises `TypeError` for non-string input. The implementation may use Python standard-library modules.
