# Specification: `top_words`

## Interface

Implement `top_words(text)` in `top_words.py` for Python 3. The argument must satisfy `isinstance(text, str)`; instances of subclasses of `str` are accepted. Any other value raises `TypeError` and is not converted to a string.

The result is a new Python `list` of zero to ten `(word, count)` tuples. Each `word` is a string and each `count` is an integer. The list contains one tuple per distinct word.

## Word and counting rules

1. Normalize the input to Unicode NFC, apply Unicode `casefold()`, then normalize to NFC again. The Unicode character properties and case-folding mappings are those provided by the running Python interpreter's `unicodedata` database (`unicodedata.unidata_version`). Therefore, results for characters added or changed in a later Unicode version may differ between Python installations.
2. A word starts with a code point whose Unicode general category begins with `L` (a letter). After its first letter, a word may contain any number of code points whose category begins with `L` or `M` (a combining mark). A mark belongs to the preceding word and cannot start a word by itself. Any other code point ends the word and acts as a separator.
3. This category rule determines how apostrophe-like characters behave. Apostrophes in punctuation categories, such as U+0027 APOSTROPHE (`'`) and U+2019 RIGHT SINGLE QUOTATION MARK (`’`), are separators. U+02BC MODIFIER LETTER APOSTROPHE (`ʼ`) has category `Lm`, so it is a letter under this rule and remains inside a word (for example, `canʼt` is one word). The Unicode category rule takes precedence over how a character looks or is named.
4. The procedure groups adjacent letters and attached marks; it does not perform language-aware word segmentation. Thus, a sequence of Chinese or Japanese characters without separators is one word under these rules.
5. Count each resulting word exactly as it appears after the normalization and case folding in rule 1. Beyond those steps, do not remove accents, stem words, or otherwise alter spelling. Case folding may change a word's length or spelling (for example, `Straße` becomes `strasse`).
6. Order results by count descending. When counts are equal, order words by ascending Python string order (lexicographic Unicode code point sequence).
7. Return the first ten entries after sorting, or all entries if fewer than ten distinct words occur. Return an empty list if no words occur.

## Examples

In the examples, `\u0301` denotes the single combining acute accent code point U+0301. The visible `é` in `café` is the precomposed character U+00E9.

| Input (Python value) | Expected output | Decision illustrated |
| --- | --- | --- |
| `'Cat cat DOG!'` | `[('cat', 2), ('dog', 1)]` | Case folding combines upper- and lowercase occurrences; punctuation separates words; fewer than 10 distinct words are returned. |
| `'go2go'` | `[('go', 2)]` | A digit separates adjacent letter sequences. |
| `'q\u0301q'` | `[('q́q', 1)]` | A combining mark following a letter remains attached to that word, so the sequence is one word. |
| `42` | raises `TypeError` | A non-string input is rejected rather than converted. |
| `('word',)` | raises `TypeError` | A non-string container is rejected. |
| `MyStr('Word')` where `MyStr` subclasses `str` | `[('word', 1)]` | Instances of `str` subclasses are accepted. |
| "can't can’t" | `[('can', 2), ('t', 2)]` | U+0027 and U+2019 are punctuation separators; the equal counts use string ordering. |
| `'canʼt'` | `[('canʼt', 1)]` | U+02BC has letter category `Lm`, so it stays inside the word under the category rule. |
| 'Мир мирный МИР' | `[('мир', 2), ('мирный', 1)]` | Cyrillic letters are words and case folding is applied. |
| 'ҚАЗАҚ қазақ' | `[('қазақ', 2)]` | Kazakh Cyrillic letters are words and case folding is applied. |
| `'Cafe\u0301 café'` | `[('café', 2)]` | NFC makes decomposed and precomposed accented spellings equivalent. |
| `'İstanbul İstanbul'` | `[('i̇stanbul', 2)]` | The combining dot produced by case folding stays attached to the initial letter. |
| 'हिन्दी' | `[('हिन्दी', 1)]` | Combining marks used in Devanagari stay attached to their preceding letters. |
| '東京は大きい' | `[('東京は大きい', 1)]` | No language-aware segmentation is performed; adjacent letters form one word. |
| 'Straße STRASSE' | `[('strasse', 2)]` | Case folding can change word length and merge spellings. |
| `'z a'` | `[('a', 1), ('z', 1)]` | Equal counts use Python string order, not first occurrence. |
| `''` | `[]` | Empty input has no words. |
| `' \t\n'` | `[]` | Whitespace-only input has no words. |
| '123 - ... 🙂' | `[]` | Digits, punctuation, and symbols alone do not form words. |
| 'a a a a b b b c c d d e e f f g h i j k l' | `[('a', 4), ('b', 3), ('c', 2), ('d', 2), ('e', 2), ('f', 2), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | Sort by count first, then by string order; the tie at the tenth position is resolved before truncation, so `k` and `l` are omitted. |

`MyStr` in the subclass example means a user-defined class declared as `class MyStr(str): pass`.
