# Top words — Specification

A news site shows a "Trending words" box beside each article. `top_words` finds the 10 most common words in an article's text, together with how many times each word appears. Articles may be in English, Russian, or Kazakh.

## Interface

- Python 3, in a file `top_words.py`.
- Signature:

  ```python
  def top_words(text: str) -> list[tuple[str, int]]
  ```

- `text` is the article text (a `str` that may contain newlines, punctuation, digits, emoji, and text in any language).
- The return value is a list of up to 10 `(word, count)` pairs, where `word` is a `str` and `count` is an `int ≥ 1`.
- The list is sorted by `count` descending; pairs with equal counts are sorted alphabetically (by Unicode code-point order) by `word`.
- Every returned `word` is lowercase.
- If the text has fewer than 10 distinct words, the list has fewer than 10 pairs (possibly none). The list is never padded.
- If `text` contains no words at all, the result is `[]`.

## Definition of a word

1. Convert the entire text to lowercase with `str.lower()`.
2. A word is a maximal run of Unicode letters (any script: Latin, Cyrillic, etc.). Every word therefore starts and ends with a letter.
3. An apostrophe — `'` (U+0027) or `’` (U+2019) — is part of a word only when it lies between two letters of the same run; anywhere else it is a separator.
4. Every other character — whitespace, digits, punctuation, emoji, symbols — is a separator and can never appear inside a word.
5. No normalization beyond lowercasing: no stemming, no stop-word removal, no folding of `ё` into `е`. Every distinct word counts as itself.

## Decisions and their examples

Each example below has a different output under at least one other plausible reading of the original problem statement.

### 1. Counting ignores case; the output word is lowercase

| Input | Expected output |
| --- | --- |
| `Dog dog DOG cat` | `dog: 3, cat: 1` |

Case-sensitive counting would instead give `dog: 2, DOG: 1, cat: 1`.

### 2. Punctuation and hyphens split words; words are not stemmed

| Input | Expected output |
| --- | --- |
| `state-of-the-art report, report!` | `report: 2, art: 1, of: 1, state: 1, the: 1` |

A reading that keeps hyphenated words whole would instead give `report: 2, state-of-the-art: 1`.

### 3. An internal apostrophe stays in the word; leading/trailing apostrophes do not

| Input | Expected output |
| --- | --- |
| `don't don't dont` | `don't: 2, dont: 1` |
| `'tis 'tis` | `tis: 2` |

Treating every apostrophe as a separator would instead give `don: 2, t: 2, dont: 1` for the first row.

### 4. Digits are separators, so digit–letter boundaries split words, and pure numbers are not words

| Input | Expected output |
| --- | --- |
| `COVID19 covid 19` | `covid: 2` |

A reading in which digits belong to words would instead give `covid19: 1, covid: 1` (and possibly `19: 1`).

### 5. Equal counts are ordered alphabetically, not by first occurrence

| Input | Expected output |
| --- | --- |
| `b a a b` | `a: 2, b: 2` |

Breaking ties by first occurrence would instead give `b: 2, a: 2`.

### 6. Russian Cyrillic works; `ё` is its own letter and is not folded into `е`

| Input | Expected output |
| --- | --- |
| `кот пёс кот кот пёс` | `кот: 3, пёс: 2` |
| `Ёж ёж еж` | `ёж: 2, еж: 1` |

Folding `ё` into `е` would instead give `еж: 3` in the second row.

### 7. Kazakh letters (ә ғ қ ң ө ұ ү һ і) are letters, and their uppercase forms fold into the same word

| Input | Expected output |
| --- | --- |
| `Қала қала үй` | `қала: 2, үй: 1` |

An ASCII-only definition of "letter" would instead give `[]` (no words found at all).

### 8. At most 10 pairs are returned; extra distinct words are dropped

The input `a b c d e f g h i j k l` (twelve distinct words, each once) returns exactly:

| Input | Expected output |
| --- | --- |
| `a b c d e f g h i j k l` | `a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1, j: 1` |

A reading that returns all distinct words would instead return 12 pairs (`… j: 1, k: 1, l: 1`).

### 9. Fewer than 10 distinct words returns a shorter list

| Input | Expected output |
| --- | --- |
| `one two` | `one: 1, two: 1` |

A reading that pads the result to 10 pairs would instead return `one: 1, two: 1, …, eighth filler: 0` (8 extra entries).

### 10. Text with no words returns an empty list

| Input | Expected output |
| --- | --- |
| `""` | `[]` |
| `123 ...!!! 😊` | `[]` |

A reading that returns `None` or raises an exception for such text would differ.

## Reference run

With `text = "Dog dog DOG cat"`, `top_words(text)` must equal `[("dog", 3), ("cat", 1)]` — the exact value, so that pairs are tuples in this order with an `int` count.
