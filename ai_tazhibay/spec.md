# Top words: specification

## Purpose

Given a text, report the ten words that occur most often in it, together with how many times each occurs. The text may mix English, Russian, and Kazakh. Any two implementations that follow this specification on the same Python version must return the same result for the same input.

## Interface

```python
def top_words(text: str) -> list[tuple[str, int]]: ...
```

- `text` must be an instance of `str` (subclasses allowed). For any other type, including `bytes` and `None`, raise `TypeError`.
- The function returns a new list of `(word, count)` tuples. It does not print, read files, or modify global state.
- The list has at most 10 entries. If the text contains fewer than 10 distinct words, return all of them. If it contains no words, return `[]`.

## Definitions

Character categories come from `unicodedata.category()` of the running Python interpreter.

- **Letter**: a character whose category starts with `L` (for example `a`, `ж`, `ә`, `ү`).
- **Mark**: a character whose category starts with `M` (combining accents such as U+0301).
- **Apostrophe**: exactly U+0027 `'` or U+2019 `’`.

## Algorithm

1. **Normalize the text.** Apply `unicodedata.normalize("NFC", text)`, then `str.lower()` (not `str.casefold()`, so `ß` stays `ß`). Work only on this normalized text from here on.
2. **Split into words.** Scan the normalized text from left to right. A word:
   - starts at a letter;
   - continues while the next character is a letter or a mark;
   - may contain an apostrophe only when the character directly before it is a letter or mark of the current word **and** the character directly after it is a letter. A word never starts or ends with an apostrophe.

   Every other character (spaces, digits, `-`, `_`, other punctuation, symbols, emoji, a mark that is not inside a word) ends the current word and is not part of any word.
3. **Unify apostrophes.** Inside each word, replace `’` with `'`.
4. **Count.** Count how many times each distinct word appears. Words are compared as exact strings: no stemming, no stop-word removal, no merging of similar letters (`е` ≠ `ё`, Latin `a` ≠ Cyrillic `а`).
5. **Sort and cut.** Sort by count, highest first. Break ties by the word in ascending Python string order (Unicode code point order). Return the first 10 entries.

## Examples

Inputs and outputs are Python literals.

| # | Input | Output | Shows |
|---|---|---|---|
| 1 | `"the cat and the dog and the bird"` | `[('the', 3), ('and', 2), ('bird', 1), ('cat', 1), ('dog', 1)]` | counting, tie-break |
| 2 | `"Hello HELLO hello"` | `[('hello', 3)]` | case is ignored |
| 3 | `"Мен қазақпын. Мен студентпін!"` | `[('мен', 2), ('студентпін', 1), ('қазақпын', 1)]` | Kazakh, code point order (`с` U+0441 < `қ` U+049B) |
| 4 | `"Привет, привет, мир"` | `[('привет', 2), ('мир', 1)]` | Russian, punctuation |
| 5 | `"ёлка елка"` | `[('елка', 1), ('ёлка', 1)]` | `ё` is not `е` |
| 6 | `"don't don’t 'quoted'"` | `[("don't", 2), ('quoted', 1)]` | apostrophes |
| 7 | `"well-known"` | `[('known', 1), ('well', 1)]` | hyphen splits |
| 8 | `"abc123def x_y"` | `[('abc', 1), ('def', 1), ('x', 1), ('y', 1)]` | digits and `_` split |
| 9 | `"é é"` | `[('é', 2)]` | NFC joins `e` + accent |
| 10 | `"a b c d e f g h i j k"` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | limit of 10 |
| 11 | `"k k a b c d e f g h i j"` | `[('k', 2), ('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1)]` | count beats alphabet |
| 16 | `"ҚАЗАҚ қазақ Қазақ"` | `[('қазақ', 3)]` | Kazakh upper/lower case |
| 17 | `"Straße STRASSE"` | `[('strasse', 1), ('straße', 1)]` | `.lower()`, not `.casefold()` |
| 12 | `""` | `[]` | empty text |
| 13 | `"123 ... !!!"` | `[]` | no words |
| 14 | `None` | raises `TypeError` | wrong type |
| 15 | `b"cat"` | raises `TypeError` | bytes are not text |

## Not covered

There is no limit on input length, and any algorithm that gives these results is acceptable.
