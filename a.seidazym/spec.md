# Top Words — Formal Specification (revised)

## 1. Purpose and scope

Implement a Python function that returns the ten most frequent words in an article. The input may contain English, Russian, and Kazakh (Cyrillic or Latin) text, including mixed-language text. This is a *frequency* list, not a list of informative keywords: common stop words such as `the`, `and`, `и`, and `в` are counted and may dominate. No stop-word filtering, stemming, lemmatization, translation, or locale-specific spelling equivalence is performed.

Kazakh Cyrillic and Latin text is supported as Unicode text. The 2017 apostrophe-based Kazakh Latin orthography is not interpreted linguistically: apostrophes are treated as internal connectors only when between letters, so its intended letter equivalences are **not** reconstructed. The output makes no claim of linguistic equivalence between scripts.

## 2. Interface and environment

- File: `top_words.py`; function: `top_words(text)`.
- Input: any instance of `str`, including a subclass (`isinstance(text, str)` is true). Other types raise `TypeError`.
- Return: `list[tuple[str, int]]` with up to ten pairs `(normalized_word, frequency)`; frequencies are positive integers.
- The function must not print, write files, mutate global state, or produce other externally observable side effects.
- Reference behavior is defined for **CPython 3.14.6 with Unicode 16.0**. Unicode character properties, NFC, and case folding depend on the Unicode database. Other Python/Unicode versions may differ on some characters; identical output is guaranteed only when the Unicode data and the rules below agree.

## 3. Normalization and word boundaries

1. Normalize the full input to Unicode NFC (`unicodedata.normalize('NFC', text)`).
2. Scan left to right to identify words **before** case folding. A word begins with a Unicode alphabetic character (`ch.isalpha()`). Once a word begins, subsequent alphabetic characters and combining marks (`unicodedata.category(ch).startswith('M')`) belong to that word. An isolated combining mark does not begin a word and is discarded as a separator.
3. The ASCII apostrophe `'` (U+0027), curly apostrophe `’` (U+2019), and ASCII hyphen `-` (U+002D) are included in a word **only when immediately preceded and immediately followed by alphabetic characters in the NFC-normalized input**. A connector elsewhere separates words. Other punctuation, digits, symbols, whitespace, and underscores separate words. Combining marks do not count as alphabetic for the immediate-neighbor connector rule.
4. For each extracted word, apply `word.casefold()` and then NFC normalization again. Count by this normalized key, and return that key, **not** the original spelling. Word boundaries are fixed first because case folding can expand or change characters.

Consequences: `Straße` is displayed as `strasse`; Greek final sigma `ς` folds to `σ`. Russian `ё` and `е` remain different. Turkish `İ` case-folds to `i` plus a combining dot and remains distinct from plain `i` after NFC. These are intentional consequences of Unicode case folding rather than language-specific correction.

## 4. Counting and ranking

Count every occurrence across the entire input. Sort pairs first by **descending frequency**, then by **ascending normalized Unicode code-point string order** (Python's default string comparison). Return the first ten pairs. If there are fewer than ten unique keys, return all; if there are none, return `[]`. Do not use locale-aware sorting. Ties at the tenth position are broken by the same string-order rule.

## 5. Examples (exact expected results)

```python
assert top_words('cat dog cat bird dog cat') == [('cat', 3), ('dog', 2), ('bird', 1)]
assert top_words('Hello hello HELLO World') == [('hello', 3), ('world', 1)]
assert top_words('hello,world! hello.world') == [('hello', 2), ('world', 2)]
assert top_words('banana apple cherry') == [('apple', 1), ('banana', 1), ('cherry', 1)]
assert top_words('Привет мир привет МИР') == [('мир', 2), ('привет', 2)]
assert top_words('Сәлем әлем сәлем ӘЛЕМ') == [('сәлем', 2), ('әлем', 2)]
assert top_words('cat123dog cat-dog') == [('cat', 1), ('cat-dog', 1), ('dog', 1)]
assert top_words('') == []
assert top_words('123 !!! 456') == []
assert top_words('a b c d e f g h i j k l') == [(x, 1) for x in 'abcdefghij']
assert top_words('the cat and the dog') == [('the', 2), ('and', 1), ('cat', 1), ('dog', 1)]
assert top_words("don't well-known") == [("don't", 1), ('well-known', 1)]
assert top_words('café cafe\u0301') == [('café', 2)]
assert top_words('Straße STRASSE') == [('strasse', 2)]
assert top_words(' \n\t ') == []
assert top_words('a\u0301 a\u0301') == [('á', 2)]
assert top_words("it's it's what's") == [("it's", 2), ("what's", 1)]
assert top_words('что-то кое-как') == [('кое-как', 1), ('что-то', 1)]
assert top_words("it’s it's") == [("it's", 1), ('it’s', 1)]
assert top_words('ёлка елка') == [('елка', 1), ('ёлка', 1)]
assert top_words('İstanbul ISTANBUL') == [('istanbul', 1), ('i\u0307stanbul', 1)]
assert top_words('\u0301') == []
assert top_words('x\u0301\u0301') == [('x\u0301\u0301', 1)]
assert top_words('cat_dog') == [('cat', 1), ('dog', 1)]
assert top_words('a- -b a--b') == [('a', 2), ('b', 2)]
```

Apostrophe forms `'` and `’` are **not** mapped to each other; they remain different normalized keys. `\u0301` is the combining acute accent. Example results are illustrative; the rules in §§2–4 govern inputs not shown here.

## 6. Errors, complexity, and acceptance

`top_words(None)`, `top_words(123)`, `top_words(['cat'])`, and `top_words(b'cat')` must raise `TypeError`. A `str` subclass must be accepted. The function must process the entire input without an arbitrary size cutoff. The expected approach takes O(n + u log u) time for input length n and u unique normalized words, with O(n + u) auxiliary storage (excluding the returned list); this is a performance target rather than an exact memory cap. A correct implementation follows all rules, not merely the examples.
