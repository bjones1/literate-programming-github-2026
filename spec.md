Specification: `top_words`
==========================

1\. Purpose
-----------

Implement a Python function that analyzes an article written in English,
Russian, or Kazakh and returns its 10 most frequent words, along with their
occurrence counts.

The function must:

* Count words without distinguishing uppercase and lowercase forms.
* Support English, Russian, and Kazakh letters, including Kazakh-specific
  letters.
* Handle Unicode text safely and consistently.
* Define explicit behavior for punctuation, digits, whitespace, invisible
  characters, and malformed or unexpected inputs.
* Return deterministic results, including when multiple words have the same
  frequency.

The function does not need to determine the article's language. It applies the
same Unicode-based rules to every input.

2\. Function interface
----------------------

<strong>Module:</strong> `top_words.py`

<strong>Function:</strong>

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

Requirements:

* Python 3.9 or later.
* Standard library only.
* No external services or packages.
* Input must be a Python string.
* Output must be a list of `(word, count)` tuples.

3\. Input validation
--------------------

The function accepts a `str` instance, including a subclass of `str`.

If `text` is not a string, raise:

```python
TypeError("text must be a str")
```

Do not convert the input to a string automatically.

| Input                          | Expected behavior |
| ------------------------------ | ----------------- |
| `"Hello world"`                | Process normally  |
| `""`                           | Return `[]`       |
| `"   "`                        | Return `[]`       |
| `None`                         | Raise `TypeError` |
| `123`                          | Raise `TypeError` |
| `3.14`                         | Raise `TypeError` |
| `True` or `False`              | Raise `TypeError` |
| `b"Hello"`                     | Raise `TypeError` |
| `["hello", "world"]`           | Raise `TypeError` |
| `{"article": "hello"}`         | Raise `TypeError` |
| An arbitrary non-string object | Raise `TypeError` |
| A subclass of `str`            | Process normally  |

The exception message must be exactly `text must be a str`.

Do not catch and suppress this validation error. Other genuine errors caused by
resource exhaustion may propagate.

4\. Input length
----------------

There is no arbitrary minimum or maximum input length.

* A string of one character is valid.
* A string of exactly 10 Unicode code points is valid.
* Empty strings are valid inputs.
* Long articles must be processed in full, subject to available system
  resources.
* The function must not stop processing when it encounters the 11th unique word.
* The limit of 10 applies only to the number of returned results.

| Input           | Expected result                                      |
| --------------- | ---------------------------------------------------- |
| `"a"`           | `[("a", 1)]`                                         |
| `"abcdefghij"`  | `[("abcdefghij", 1)]`                                |
| `"a b c d e"`   | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1)]` |
| `"hello hello"` | `[("hello", 2)]`                                     |
| `"😀😀😀"`      | `[]`                                                 |

A Python string's length is measured in Unicode code points, not necessarily
visible characters. For example, an emoji sequence may contain multiple code
points. The function does not need to count visible grapheme clusters.

5\. Unicode normalization
-------------------------

Before tokenization, normalize the entire input using Unicode NFC:

```python
text = unicodedata.normalize("NFC", text)
```

For each completed token:

1. Apply Python's `casefold()`.
2. Normalize the result using NFC again.
3. Use the resulting string as the counting key.

Do not use NFKC or NFKD. Do not remove accents, transliterate letters, stem
words, or remove stop words.

Canonically equivalent spellings should count together when NFC makes them
equivalent.

| Input               | Expected result  |
| ------------------- | ---------------- |
| `"café cafe\u0301"` | `[("café", 2)]`  |
| `"ҚАЗАҚ қазақ"`     | `[("қазақ", 2)]` |
| `"МӘТІН мәтін"`     | `[("мәтін", 2)]` |

6\. Definition of a word
------------------------

A word is a maximal sequence of permitted Unicode characters.

### Characters that may start or continue a word

Accept characters with these Unicode general categories:

* `Lu` — uppercase letters
* `Ll` — lowercase letters
* `Lt` — titlecase letters
* `Lm` — modifier letters
* `Lo` — other letters
* `Nd` — decimal digits

This includes English, Russian, and Kazakh letters, as well as letters from
other Unicode scripts.

### Combining marks

Characters with categories `Mn`, `Mc`, or `Me` may continue an existing word but
cannot start a word.

* A combining mark following a word is included in that token.
* A combining mark encountered when no token is active is ignored.
* Combining marks are not counted as independent words.

### Other characters

Every character outside the permitted categories ends the current word and is
excluded from the token.

Examples include whitespace, punctuation, symbols, emoji, control characters,
and Unicode format characters.

No dictionary, language detector, stemming algorithm, or language-specific
segmentation is used.

| Input         | Expected result              |
| ------------- | ---------------------------- |
| `"hello-мир"` | `[("hello", 1), ("мир", 1)]` |
| `"abcабв"`    | `[("abcабв", 1)]`            |
| `"қазақ2026"` | `[("қазақ2026", 1)]`         |
| `"你好世界"`  | `[("你好世界", 1)]`          |
| `"a\u0301 a"` | `[("a", 1), ("á", 1)]`       |
| `"\u0301a a"` | `[("a", 2)]`                 |

<strong>Scope note:</strong> This is a Unicode-character tokenizer, not a full
linguistic tokenizer. For example, a Chinese sentence without separators is
treated as one token. That behavior is outside the primary English, Russian, and
Kazakh use case.

7\. Punctuation and separators
------------------------------

Punctuation separates words. It must not be deleted in a way that joins the
surrounding text, and it must never appear in the output.

Treat all punctuation characters as separators, including:

* Commas and periods
* Colons and semicolons
* Straight and curly quotation marks
* Straight and curly apostrophes
* Hyphens and dashes
* Parentheses and brackets
* Underscores and slashes

Repeated punctuation does not create empty words. Leading and trailing
punctuation is ignored.

### Commas

| Input          | Expected result             |
| -------------- | --------------------------- |
| `"red,blue"`   | `[("blue", 1), ("red", 1)]` |
| `"red,,,blue"` | `[("blue", 1), ("red", 1)]` |
| `"a,b,a"`      | `[("a", 2), ("b", 1)]`      |
| `"1,000"`      | `[("000", 1), ("1", 1)]`    |

### Apostrophes

Both straight (`'`) and curly (`’`) apostrophes are separators.

| Input           | Expected result                        |
| --------------- | -------------------------------------- |
| `"don't"`       | `[("don", 1), ("t", 1)]`               |
| `"don't don't"` | `[("don", 2), ("t", 2)]`               |
| `"rock’n’roll"` | `[("n", 1), ("rock", 1), ("roll", 1)]` |
| `"James' book"` | `[("book", 1), ("james", 1)]`          |

### Hyphens and dashes

ASCII hyphens (`-`), en dashes (`–`), em dashes (`—`), and other punctuation
dash characters are separators.

| Input            | Expected result                  |
| ---------------- | -------------------------------- |
| `"well-known"`   | `[("known", 1), ("well", 1)]`    |
| `"well--known"`  | `[("known", 1), ("well", 1)]`    |
| `"well---known"` | `[("known", 1), ("well", 1)]`    |
| `"a-b-c"`        | `[("a", 1), ("b", 1), ("c", 1)]` |
| `"hello—world"`  | `[("hello", 1), ("world", 1)]`   |
| `"---"`          | `[]`                             |

The function does not replace punctuation in the original input. It simply uses
punctuation to determine token boundaries.

8\. Whitespace and invisible characters
---------------------------------------

All whitespace characters act as separators, including ordinary spaces, tabs,
and newlines.

Control and format characters also act as separators. In particular:

* Zero-width space (`U+200B`) is a separator.
* Zero-width joiner (`U+200D`) is a separator.
* Byte order mark (`U+FEFF`), when present in the string, is a separator.
* Null character (`U+0000`) is a separator.
* Private-use and unassigned code points are separators.

Combining marks are governed by Section 6 and must not be indiscriminately
stripped.

| Input                   | Expected result                |
| ----------------------- | ------------------------------ |
| `"hello world"`         | `[("hello", 1), ("world", 1)]` |
| `"hello\tworld\nhello"` | `[("hello", 2), ("world", 1)]` |
| `"hel\u200Blo"`         | `[("hel", 1), ("lo", 1)]`      |
| `"hel\u200Dlo"`         | `[("hel", 1), ("lo", 1)]`      |
| `"\uFEFFhello\uFEFF"`   | `[("hello", 1)]`               |
| `"a\u0000b"`            | `[("a", 1), ("b", 1)]`         |
| `"a\uE000b"`            | `[("a", 1), ("b", 1)]`         |
| `"★ 😀 ♫"`              | `[]`                           |

9\. Digits and numbers
----------------------

Unicode decimal digits in category `Nd` may be part of a word. Digits from
different scripts are not converted to ASCII.

Characters in categories `Nl` and `No`, such as Roman numeral symbols,
superscript digits, circled digits, and fractions, are separators.

| Input             | Expected result               |
| ----------------- | ----------------------------- |
| `"abc123 abc123"` | `[("abc123", 2)]`             |
| `"3.14"`          | `[("14", 1), ("3", 1)]`       |
| `"2026 2025"`     | `[("2025", 1), ("2026", 1)]`  |
| `"１２３ 123"`    | `[("123", 1), ("１２３", 1)]` |
| `"١٢٣ 123"`       | `[("123", 1), ("١٢٣", 1)]`    |
| `"² ½ ①"`         | `[]`                          |

10\. Case-insensitive counting
------------------------------

Use Python's `str.casefold()` rather than `lower()`.

Do not apply locale-specific rules for English, Russian, or Kazakh. Follow
Python's Unicode case-folding behavior.

| Input                 | Expected result    |
| --------------------- | ------------------ |
| `"Apple apple APPLE"` | `[("apple", 3)]`   |
| `"HELLO Hello hello"` | `[("hello", 3)]`   |
| `"Straße STRASSE"`    | `[("strasse", 2)]` |
| `"ҚАЗАҚ Қазақ қазақ"` | `[("қазақ", 3)]`   |

Characters that look similar but are not equivalent under NFC and case folding
remain separate. For example, Latin `a` and Cyrillic `а` must not be merged.

11\. Literal escape sequences
-----------------------------

The function processes the actual characters in the input string. It does not
decode textual escape sequences.

| Python expression | Expected result              |
| ----------------- | ---------------------------- |
| `"\u00E9"`        | `[("é", 1)]`                 |
| `r"\u00E9"`       | `[("u00e9", 1)]`             |
| `"\U0001F600"`    | `[]`                         |
| `r"\U0001F600"`   | `[("u0001f600", 1)]`         |
| `r"\N{SNOWMAN}"`  | `[("n", 1), ("snowman", 1)]` |

The raw-string examples contain literal backslashes; they do not contain the
characters represented by those escape sequences.

12\. Counting and sorting
-------------------------

Count all tokens in the complete input before limiting the results.

Sort by these rules, in order:

1. Frequency, descending.
2. Final normalized word, ascending by Unicode code-point order.

Do not use locale-aware sorting, numeric sorting, or first-occurrence order as a
tie-breaker.

| Input         | Expected result                  |
| ------------- | -------------------------------- |
| `"b a c"`     | `[("a", 1), ("b", 1), ("c", 1)]` |
| `"z a z b a"` | `[("a", 2), ("z", 2), ("b", 1)]` |
| `"2 10"`      | `[("10", 1), ("2", 1)]`          |

After sorting, return the first 10 entries only.

* If there are fewer than 10 unique words, return all of them.
* If there are more than 10 unique words, return exactly 10.
* Do not include an `"other"` entry.
* Do not report the number of omitted words.
* Do not stop counting after encountering 10 unique words.

13\. Output contract
--------------------

Return a list of tuples in this format:

```python
[
    ("word", 5),
    ("another", 3),
]
```

Each tuple contains:

* A nonempty, NFC-normalized, case-folded word.
* A positive integer count.

The output must:

* Contain no more than 10 tuples.
* Contain no duplicate normalized words.
* Follow the required sorting order.
* Be `[]` if no words are found.

Do not return a dictionary, set, generator, JSON string, or list of
dictionaries.

14\. Performance and reliability
--------------------------------

* Process the complete input, subject to available resources.
* Do not impose an arbitrary input-length limit.
* Do not mutate the input.
* Do not depend on external packages or services.
* Use Unicode general categories as defined by Python's `unicodedata` module.
* Expect counting to be linear in input length, apart from normalization costs,
  followed by sorting the unique tokens.

15\. Acceptance checklist
-------------------------

An implementation is correct only if it:

1. Rejects non-string inputs with the exact required `TypeError`.
2. Accepts empty strings, short strings, and strings of 10 or more code points.
3. Normalizes text with NFC before tokenization.
4. Uses `casefold()` and NFC on every completed token.
5. Implements the exact Unicode-category rules for letters, digits, and
   combining marks.
6. Treats punctuation, whitespace, control characters, and format characters as
   separators.
7. Handles commas, apostrophes, hyphens, and repeated dashes as specified.
8. Handles invisible characters without accidentally joining adjacent words.
9. Distinguishes visually similar but non-equivalent Unicode characters.
10. Counts all tokens before selecting the top 10.
11. Applies deterministic frequency and code-point sorting.
12. Returns only unique words with positive integer counts.
13. Does not decode literal escape sequences.
14. Does not alter the input or suppress genuine resource errors.
