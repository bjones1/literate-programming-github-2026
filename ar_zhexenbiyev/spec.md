# Specification: `top_words`

## Purpose

`top_words` extracts the ten most frequent words from one article. It supports
Unicode text, including English, Russian, and Kazakh. The function is
deterministic: the same input always produces the same result.

## Interface

Create `top_words.py` containing:

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

The function accepts exactly one argument, `text`.

- If `text` is not a `str`, raise `TypeError`.
- Do not mutate the input.
- For an empty string or text containing no words, return `[]`.
- A result has at most ten entries, and has fewer than ten entries when fewer
  than ten distinct words occur.
- Each tuple contains the normalized word and its positive occurrence count.

## Unicode and normalization

1. Normalize the input with Unicode NFC before finding words. Thus canonically
   equivalent spellings count as the same word. After extracting and
   normalizing a word, apply `casefold()` and then NFC again to the resulting
   output word.
2. A character means one Unicode code point, not one UTF-8 byte and not one
   user-perceived grapheme cluster. A combining mark that remains after NFC is
   not a letter for this specification and is therefore a separator.
3. A word is a maximal run of Unicode letters (`str.isalpha()` is true),
   optionally containing internal apostrophes. The apostrophe characters
   recognized for this purpose are ASCII apostrophe (`'`), right single
   quotation mark (`’`), and modifier letter apostrophe (`ʼ`).
4. An apostrophe belongs to a word only when it has a letter immediately on
   both sides. Therefore leading, trailing, and repeated apostrophes are
   separators. Normalize every recognized internal apostrophe to ASCII
   apostrophe in the output before case folding, so visually equivalent
   apostrophe forms count together.
5. Digits, underscores, emoji, symbols, punctuation, and all whitespace are
   separators. In particular, a number such as `2026` is not a word.
6. Hyphens and dashes are separators, so `well-being` yields `well` and
   `being`, and `қазақ-тілі` yields `қазақ` and `тілі`.
7. Lowercase every extracted word with Unicode `str.casefold()`. English,
   Russian, and Kazakh case distinctions therefore do not create separate
   entries. The implementation must use Python's standard-library
   `str.isalpha()`, `str.casefold()`, and `unicodedata.normalize("NFC", ...)`
   semantics; it must not substitute ASCII-only or locale-specific rules.

The following table fixes the tokenization decisions:

| Input | Words and counts |
| --- | --- |
| `cat dog cat bird dog cat` | `cat: 3`, `dog: 2`, `bird: 1` |
| `Cat CAT cAt` | `cat: 3` |
| `кот КОТ қасық ҚАСЫҚ` | `кот: 2`, `қасық: 2` |
| `well-being well’being` | `well: 2`, `being: 2` |
| `O'Reilly O’Reilly` | `o'reilly: 2` |
| `'word' __ 2026` | `word: 1` |
| `Привет, әлем! 😊` | `привет: 1`, `әлем: 1` |
| `é é` | `é: 2` |

Input validation and empty results are also part of the interface:

| Input | Expected result |
| --- | --- |
| `""` | `[]` |
| `"123 __ 😊"` | `[]` |
| `None` | raises `TypeError` |

## Ranking and ties

Count every occurrence of each normalized word. Sort entries by:

1. descending count; then
2. ascending Unicode code-point order of the normalized word.

The second rule is the complete tie-breaker; do not use input order, locale
collation, or an arbitrary dictionary order. Return only the first ten entries
after sorting.

Examples:

| Input | Expected output |
| --- | --- |
| `alpha beta beta gamma gamma alpha delta` | `[("alpha", 2), ("beta", 2), ("gamma", 2), ("delta", 1)]` |
| `z z a a m m b` | `[("a", 2), ("m", 2), ("z", 2), ("b", 1)]` |
| `one two three` | `[("one", 1), ("three", 1), ("two", 1)]` |

The ten-word limit is inclusive:

| Input | Expected output |
| --- | --- |
| `w1 w2 w3 w4 w5 w6 w7 w8 w9 w10 w11` | `[]` |

The preceding example contains no words because digits are separators. A
valid ten-plus-word boundary example is:

| Input | Expected output |
| --- | --- |
| `a b c d e f g h i j k` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` |

## Non-goals

The function does not detect language, translate words, remove stop words,
stem or lemmatize words, perform locale-specific collation, or return
percentages. It reports lexical occurrences exactly according to the rules
above.
