
# Specification: `top_words`

## 1. Original Problem Statement
Quoted from the e-book:
> "A news site shows a 'Trending words' box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh."

---

## 2. Overview and Interface

- **Language requirement**: Python 3.9+ (utilizing built-in generic type hints `list[tuple[str, int]]`).
- **Module & Function**: Module `top_words.py`, function `top_words`.

```python
def top_words(text: str) -> list[tuple[str, int]]:
    """Return the top 10 most frequent words in `text` and their counts."""

Return Contract

The function returns a list of (word, count) tuples:

  - word (str): Normalized lowercase word string.
  - count (int): Exact integer number of occurrences in the text.
  - The returned list contains at most 10 elements, ordered by:
    1.  count in descending order.
    2.  word in ascending Unicode code-point order for ties.

3. Specification Decisions and Disambiguation

Decision 1: Explicit Code-Point Word Definition & Unicode Normalization

  - Normalization: The input text is first normalized to Unicode NFC
    (unicodedata.normalize('NFC', text)). This ensures decomposed character
    sequences (such as NFD е + \u0308 for ё) are composed into single code
    points before tokenization.
  - Allowed Characters: A word is a maximal contiguous sequence of characters
    belonging strictly to the following Unicode code-point ranges:
      - Basic Latin (English):
          - Uppercase A–Z: U+0041 to U+005A
          - Lowercase a–z: U+0061 to U+007A
      - Russian Cyrillic:
          - А–Я: U+0410 to U+042F
          - а–я: U+0430 to U+044F
          - Ё: U+0401, ё: U+0451
      - Kazakh Cyrillic Extensions:
          - Ә (U+04D8), ә (U+04D9)
          - Ғ (U+0492), ғ (U+0493)
          - Қ (U+049A), қ (U+049B)
          - Ң (U+04A2), ң (U+04A3)
          - Ө (U+04E8), ө (U+04E9)
          - Ұ (U+04B0), ұ (U+04B1)
          - Ү (U+04AE), ү (U+04AF)
          - Һ (U+04BA), һ (U+04BB)
          - І (U+0406), і (U+0456)
  - Mixed Scripts: Mixed-script sequences containing both Latin and Cyrillic
    allowed code points (e.g., Пpивeт using Latin p and e) form a single word
    token.
  - Disallowed Characters & Fragmentation: Any character outside these explicit
    ranges (including ASCII digits 0–9, accented Latin characters like é
    (U+00E9), and Kazakh Latin letters like ä, ş, ğ) acts as a delimiter and
    fragments words.
  - Rejected alternative: Allowing ASCII digits in words (e.g., "covid19" \to
    "covid").
  - Rejected alternative: Supporting arbitrary Unicode letters outside the
    specified languages (e.g., French "café" \to "caf").

| Input        | Expected Output                                | Plausible Alternative Output                      |
| :------------- | :------------------------------------------- | :------------------------------------------------ |
| `"Сәлем, әлем! Hello 2026 covid19 café."` | `[('caf', 1), ('hello', 1), ('covid', 1), ('сәлем', 1), ('әлем', 1)]` | `[('2026', 1), ('café', 1), ('covid19', 1), ...]` |
| `"ёж"` in NFD (`"е\u0308ж"`)              | `[('ёж', 1)]`                                                         | Without NFC: `[('е', 1), ('ж', 1)]`               |

Decision 2: Delimiters and Tokenization Precedence

  - Rule: Tokenization occurs before lowercasing. All non-allowed code points
    (including hyphens -, underscores _, and apostrophes ', ’) act strictly as
    token delimiters.
  - Compound words (e.g., Russian "кое-как", English "step-by-step") and
    contractions (e.g., "don't") are split into distinct tokens.
  - Special symbols that lowercase into allowed ASCII letters (e.g., Kelvin
    symbol K U+212A) are treated as delimiters during tokenization and are not
    converted into letters.

| Input                   | Expected Output                                      | Plausible Alternative Output          |
| :---------------------- | :---------------------------------------------------- | :------------------------------------ |
| `"кое-как don't step-by-step"`    | `[('step', 2), ('by', 1), ('don', 1), ('t', 1), ('как', 1), ('кое', 1)]` | `[("кое-как", 1), ("don't", 1), ...]` |
| `"Kelvin"` (`\u212A` + `"elvin"`) | `[('elvin', 1)]`                                                         | Lowercase first: `[('kelvin', 1)]`    |

Decision 3: Lowercasing Order and Equivalence

  - Rule: Lowercasing is applied to each token after tokenization using
    .lower(). Within the closed set of allowed code points defined in
    Decision 1, .lower() and .casefold() produce identical results.
  - Rejected alternative: Case-sensitive word matching.

| Input                        | Expected Output              | Plausible Alternative Output                   |
| :--------------------------- | :--------------------------- | :--------------------------------------------- |
| `"Алма алма АЛМА Ізде іЗдЕ"` | `[('алма', 3), ('ізде', 2)]` | `[('алма', 1), ('Алма', 1), ('АЛМА', 1), ...]` |

Decision 4: Deterministic Unicode Code-Point Tie-Breaking

  - Rule: Ties in frequency are broken strictly by ascending Unicode code point
    order (ord(c)).
  - Important distinction from locale collation: This is not linguistic
    alphabetical collation:
      - All Basic Latin letters (U+0061–U+007A) sort before Cyrillic letters
        (U+0400+).
      - Russian ё (U+0451) sorts after я (U+044F).
      - Kazakh specific Cyrillic letters (e.g., ә U+04D9) sort after Russian
        Cyrillic я (U+044F), despite sorting immediately after а in the official
        Kazakh alphabet.
  - Rejected alternative: Locale-aware alphabetical sorting (e.g., using
    locale.strcoll).

| Input              | Expected Output                           | Plausible Alternative Output                            |
| :----------------- | :---------------------------------------- | :------------------------------------------------------ |
| `"яблоко ёж әлем"` | `[('яблоко', 1), ('ёж', 1), ('әлем', 1)]` | Alphabetical: `[('ёж', 1), ('әлем', 1), ('яблоко', 1)]` |

Decision 5: No Stop-Word Filtering

  - Rule: All valid words are counted equally. No stop words are excluded in
    English, Russian, or Kazakh.
  - Tie-breaking applies code-point order: 'және' (ж = U+0436) strictly precedes
    'мысық' (м = U+043C).

| Input                     | Expected Output          | Plausible Alternative Output                             |
| :------------------- | :----------------------------- | :-------------------------------------------------- |
| `"the cat and the dog және мысық"` | `[('the', 2), ('and', 1), ('cat', 1), ('dog', 1), ('және', 1), ('мысық', 1)]` | `[('the', 2), ..., ('мысық', 1), ('және', 1)]` (wrong tie-break) |

Decision 6: Exact 10-Word Cutoff and Cutoff Ties

  - Rule: The result contains at most 10 elements.
  - If 11 or more unique words exist, only the top 10 positions are included.
    Words tied for the 10th position that sort beyond position 10 in code-point
    order are excluded (no expansion beyond 10 items).
  - If fewer than 10 unique words exist, all unique words are returned without
    padding.
  - If no valid words exist (or text is empty), returns an empty list [].

| Input                          | Expected Output                      | Plausible Alternative Output                     |
| :------------------------- | :--------------------------------------- | :----------------------------------------------- |
| `"a b c d e f g h i j k"` (11 words, count 1 each) | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | Returning all 11 tied words: includes `('k', 1)` |
| `""` or `"1234 !!!"`                               | `[]`                                                                                                   | Raises exception                                 |

Decision 7: Input Type Validation

  - Rule: The function requires isinstance(text, str). Instances of str and its
    subclasses are accepted. If text is not an instance of str (or a subclass),
    the function raises a TypeError with the exact message "Input must be a
    string".

| Input      | Expected Output                              | Plausible Alternative Output |
| :--------- | :------------------------------------------- | :--------------------------- |
| `None`     | Raises `TypeError("Input must be a string")` | Returns `[]`                 |
| `123`      | Raises `TypeError("Input must be a string")` | Converts to `"123"`          |
| `b"bytes"` | Raises `TypeError("Input must be a string")` | Decodes bytes                |


