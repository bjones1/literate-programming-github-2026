# Top words specification

Author: e_gazizuly

Source: https://github.com/bjones1/literate-programming-book/blob/main/exercises/top-words.md

## Interface

Implement `top_words(text: str) -> list[tuple[str, int]]` in `top_words.py`, using Python 3.10 or later and its standard library. The function accepts the article's complete plain text; it does not read files, fetch URLs, parse HTML, print, or modify external state. A non-string argument raises `TypeError`. String subclasses are accepted.

## Word and counting rules

1. Normalize the entire input to Unicode NFC, apply `str.casefold()`, then normalize to NFC again. All subsequent operations use this resulting string. Returned words use this normalized spelling.
2. A word is a maximal consecutive run of characters for which Python's `str.isalpha()` returns true. This includes Latin and Cyrillic letters, including Kazakh letters, and letters from other scripts. Do not detect or restrict the article's language. Unicode character classifications follow the running Python version's Unicode database.
3. Every other character is a separator, including whitespace, digits, underscores, punctuation, apostrophes, hyphens, emoji, and any combining mark remaining after normalization. Separators split words rather than being deleted. Consecutive separators produce no empty words.
4. Count each occurrence of each normalized word, including one-letter words and common words such as `the`. Do not remove stop words, stem, translate, transliterate, or merge different spellings such as Russian `е` and `ё`.
5. Sort distinct words by decreasing count. Break equal-count ties by increasing Python string order (Unicode code point order), without locale-specific collation. Apply this ordering before selecting the first ten entries, including ties at the cutoff.
6. Return a list of `(word, count)` tuples in that order. Counts are positive integers. Return fewer than ten entries if fewer than ten distinct words exist, and `[]` if there are none. There is no configurable limit.

## Acceptance examples

Inputs and outputs below use Python literal notation, including escapes. Each row is a separate call to `top_words`.

| Decision checked | Input | Expected result |
| --- | --- | --- |
| Frequency order and tuple/list format | `'cat dog cat bird dog cat'` | `[('cat', 3), ('dog', 2), ('bird', 1)]` |
| Case folding | `'Cat CAT cat'` | `[('cat', 3)]` |
| Canonical Unicode equivalence | `'café cafe\u0301'` | `[('café', 2)]` |
| Case folding can expand a letter | `'Straße STRASSE'` | `[('strasse', 2)]` |
| English, Russian, Kazakh, and mixed text | `'Cat КОТ кот ҚАЗАҚ қазақ'` | `[('кот', 2), ('қазақ', 2), ('cat', 1)]` |
| No language filter | `'猫 猫 cat'` | `[('猫', 2), ('cat', 1)]` |
| Whitespace separators | `'cat\tdog\ncat\u00a0dog'` | `[('cat', 2), ('dog', 2)]` |
| Punctuation and emoji split words | `'cat,dog.cat🙂dog'` | `[('cat', 2), ('dog', 2)]` |
| Apostrophes and hyphens split words | `"don't don’t well-known"` | `[('don', 2), ('t', 2), ('known', 1), ('well', 1)]` |
| Digits and underscores split words | `'abc123def abc_def 123'` | `[('abc', 2), ('def', 2)]` |
| Remaining combining marks split words | `'a\u0338b ab'` | `[('a', 1), ('ab', 1), ('b', 1)]` |
| Stop words and one-letter words are counted | `'the the a I'` | `[('the', 2), ('a', 1), ('i', 1)]` |
| No stemming | `'run runs running'` | `[('run', 1), ('running', 1), ('runs', 1)]` |
| Distinct Cyrillic spellings | `'елка ёлка елка'` | `[('елка', 2), ('ёлка', 1)]` |
| Code point tie ordering | `'я ә а z'` | `[('z', 1), ('а', 1), ('я', 1), ('ә', 1)]` |
| Fewer than ten distinct words | `'solo solo'` | `[('solo', 2)]` |
| Empty input | `''` | `[]` |
| No letters | `' \t\n123_!?🙂'` | `[]` |
| Ten-entry cutoff, with tied counts | `'k j i h g f e d c b a'` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` |
| Frequency outranks alphabetical position at cutoff | `'a b c d e f g h i j k k'` | `[('k', 2), ('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1)]` |
| Plain text, no HTML parsing | `'<b>cat</b>'` | `[('b', 2), ('cat', 1)]` |
| Invalid argument | `None` | Raises `TypeError` |
| No implicit conversion | `123` | Raises `TypeError` |
| No implicit decoding | `b'cat'` | Raises `TypeError` |

No timing or memory bound is required beyond producing these results for inputs that fit available memory.
