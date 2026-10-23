# Top words specification

## Purpose and interface

Use Python 3.10 or later. Define the function `top_words` in `top_words.py`:

```python
top_words(text: str) -> list[tuple[str, int]]
```

The function returns the 10 most frequent words in the input text, together with their occurrence counts.

The input must be an instance of `str`; subclasses of `str` are accepted. For any other input, raise `TypeError`. The exception message is not specified.

## Text processing

Perform these steps in order:

1. Normalize the input text using `unicodedata.normalize("NFC", text)`.
2. Extract words according to the rules below.
3. Lowercase each extracted word using Python's `str.lower()`, then normalize the result again using `unicodedata.normalize("NFC", word)`.
4. Count occurrences of the resulting strings.
5. Sort and select the results according to the rules below.

The second NFC is needed because `str.lower()` can turn an NFC string into one that is not NFC: `"J\u030C"` has no precomposed form, but its lowercase `"j\u030C"` composes to `"ǰ"` (U+01F0).

Use the Unicode data provided by the executing Python version.

### Word recognition

A letter is a character whose `unicodedata.category()` starts with `L`.

A mark is a character whose `unicodedata.category()` starts with `M`.

A word starts with a letter and continues through the maximal consecutive sequence of letters and marks.

A mark encountered when no word is being collected is ignored.

Every other character separates words. This includes:

- Spaces, tabs, and newlines.
- Digits.
- Punctuation.
- Straight and curly apostrophes.
- Hyphens and dashes.
- Underscores.
- Emoji.

Letters from all Unicode scripts are accepted, including English, Russian, and Kazakh. A change of script within a word does not split it.

### Normalization and counting

Use `str.lower()`, not `str.casefold()`.

Words are identical only when their resulting normalized strings are equal.

NFC normalization merges canonically equivalent representations. Apart from NFC normalization and lowercasing, do not transform words:

- Do not transliterate characters.
- Do not remove accents or attached marks.
- Do not stem or lemmatize words.
- Do not merge different word forms.
- Do not remove stop words.
- Do not exclude single-letter words.

For example, `cat` and `cats` remain different words. Latin `a` and Cyrillic `а` also remain different characters.

## Counting and sorting

Count every occurrence of each normalized word.

Sort the unique normalized words:

1. By descending occurrence count.
2. For equal counts, by ascending Unicode code point order using Python's standard lexicographic string comparison.

Return the first 10 entries from this sorted sequence.

If fewer than 10 unique words exist, return all entries.

Do not return extra entries when words have equal counts at the tenth position.

If no words are extracted, return `[]`.

## Output format

Return a Python list of `(str, int)` tuples.

Each tuple contains the normalized word and its positive occurrence count.

Example:

```python
[("word", 3), ("test", 1)]
```

## Examples

Rows showing `TypeError` specify an exception rather than a return value. `StrSub` is defined as `class StrSub(str): pass`.

| Input                        | Expected output                             | What it pins down                          |
|------------------------------|---------------------------------------------|--------------------------------------------|
| `""`                         | `[]`                                        | Empty input                                |
| `" \t\n"`                    | `[]`                                        | Whitespace only                            |
| `"123 !!! --- 😊"`           | `[]`                                        | No words                                   |
| `"Cat CAT cat"`              | `[("cat", 3)]`                              | English lowercasing                        |
| `"Привет привет ПРИВЕТ мир"` | `[("привет", 3), ("мир", 1)]`               | Russian lowercasing                        |
| `"Сәлем сәлем Қазақстан"`    | `[("сәлем", 2), ("қазақстан", 1)]`          | Kazakh lowercasing                         |
| `"hello,world!hello?"`       | `[("hello", 2), ("world", 1)]`              | Punctuation separates words                |
| `"don't don’t"`              | `[("don", 2), ("t", 2)]`                    | Straight and curly apostrophes             |
| `"well-known well—known"`    | `[("known", 2), ("well", 2)]`               | Hyphens and dashes separate words          |
| `"cat2dog cat_dog"`          | `[("cat", 2), ("dog", 2)]`                  | Digits and underscores separate words      |
| `"cat\tdog\ncat"`            | `[("cat", 2), ("dog", 1)]`                  | Tabs and newlines separate words           |
| `"a I the the"`              | `[("the", 2), ("a", 1), ("i", 1)]`          | All words counted; frequency first         |
| `"cat cats cat"`             | `[("cat", 2), ("cats", 1)]`                 | Word forms remain distinct                 |
| `"cafe café"`                | `[("cafe", 1), ("café", 1)]`                | Accents are preserved                      |
| `"café cafe\u0301"`          | `[("café", 2)]`                             | NFC: equivalent Latin forms                |
| `"й и\u0306"`                | `[("й", 2)]`                                | NFC: equivalent Cyrillic forms             |
| `"q\u0301 Q\u0301"`          | `[("q\u0301", 2)]`                          | Attached marks are preserved               |
| `"\u0301cat"`                | `[("cat", 1)]`                              | Leading marks are ignored                  |
| `"заме\u0301чательный"`      | `[("заме\u0301чательный", 1)]`              | Russian stress mark stays in the word      |
| `"किताब"`                    | `[("किताब", 1)]`                            | Spacing marks stay in the word             |
| `"Straße STRASSE"`           | `[("strasse", 1), ("straße", 1)]`           | lower(), not casefold()                    |
| `"ΑΣ'Α"`                     | `[("α", 1), ("ας", 1)]`                     | lower() applies to each word, not the text |
| `"İstanbul"`                 | `[("i\u0307stanbul", 1)]`                   | Marks produced by lower() stay in the word |
| `"ǰ J\u030C"`                | `[("ǰ", 2)]`                                | NFC is reapplied after lower()             |
| `"ǅ ǆ Ǆ"`                    | `[("ǆ", 3)]`                                | Titlecase letters are letters              |
| `"b a b a"`                  | `[("a", 2), ("b", 2)]`                      | Ties use lexical order                     |
| `"ё е ж"`                    | `[("е", 1), ("ж", 1), ("ё", 1)]`            | Unicode order, not dictionary order        |
| `"я қ ә а"`                  | `[("а", 1), ("я", 1), ("қ", 1), ("ә", 1)]`  | Kazakh letters use code point order        |
| `"a\u0430 a \u0430"`         | `[("a", 1), ("a\u0430", 1), ("\u0430", 1)]` | Distinct scripts; no script splitting      |
| `"cat 猫 猫"`                | `[("猫", 2), ("cat", 1)]`                   | Other Unicode scripts are accepted         |
| `StrSub("Cat cat")`          | `[("cat", 2)]`                              | `str` subclasses are accepted              |
| `None`                       | `TypeError`                                 | Null input is rejected                     |
| `123`                        | `TypeError`                                 | Numeric input is rejected                  |
| `b"cat"`                     | `TypeError`                                 | Byte strings are rejected                  |

### Ten-word limit examples

This example checks that exactly 10 entries are returned when 11 unique words have equal counts.

Input:

```python
"k j i h g f e d c b a"
```

Expected output:

```python
[
    ("a", 1),
    ("b", 1),
    ("c", 1),
    ("d", 1),
    ("e", 1),
    ("f", 1),
    ("g", 1),
    ("h", 1),
    ("i", 1),
    ("j", 1),
]
```

This example checks the same cutoff when the tied words occur more than once.

Input:

```python
"k k j j i i h h g g f f e e d d c c b b a a"
```

Expected output:

```python
[
    ("a", 2),
    ("b", 2),
    ("c", 2),
    ("d", 2),
    ("e", 2),
    ("f", 2),
    ("g", 2),
    ("h", 2),
    ("i", 2),
    ("j", 2),
]
```
