# Top words

A news site shows a "Trending words" box beside each article. `top_words`
finds the 10 most common words in an article's text, with their counts.
Articles may be in English, Russian, or Kazakh (Cyrillic, or the 2021 Latin
alphabet).

## Interface

- Python 3, standard library only. File `top_words.py`.
- `top_words(text: str) -> list[tuple[str, int]]`
- Each tuple is `(word, count)`, where `word` is a plain `str`. The list is
  sorted as in rule 9 and has at most 10 items.

## Rules

1. **Input type.** If `isinstance(text, str)` is false (including `None` and
   `bytes`), raise `TypeError`. A `str` subclass is accepted and first
   converted to a plain `str` with the same characters; none of its method
   overrides (`lower`, `__iter__`, `__getitem__`, …) are used.
2. **Normalization**, applied in this order:
   1. Delete every format character (Unicode category `Cf`), such as soft
      hyphen U+00AD, zero-width characters U+200B–U+200D, U+2060, U+FEFF, and
      direction marks U+200E/U+200F.
   2. Lowercase with `str.lower()`. This is Python's default mapping, so
      Kazakh-Latin `I` becomes `i`, not `ı`. Known limitation: all-caps words
      whose `I` means `ı` are miscounted (`ILIM` is counted as `ilim`).
   3. Apply Unicode NFC normalization (not NFKC), so fullwidth letters and
      ligatures such as `ﬁ` stay distinct from their ASCII forms.
   4. Delete every combining mark still present (Unicode category `Mn`, `Mc`
      or `Me`). Letters that NFC composed keep their accent (`й`, `é`, `ѐ`);
      marks with no precomposed form are removed (the dot `str.lower()` adds to
      `İ`, the stress mark in `'за\u0301мок'`).
   5. Replace `‘` (U+2018), `’` (U+2019) and `ʼ` (U+02BC) with ASCII `'`.
      Replace hyphen U+2010 and non-breaking hyphen U+2011 with ASCII `-`.
3. **Letter.** This and all following rules apply to the normalized text.
   A letter is any character for which `str.isalpha()` is true, including
   modifier letters (category `Lm`) such as `ˈ` U+02C8 and `ʻ` U+02BB.
4. **Word.** A word is a maximal run of letters. Each `'` or `-` that has a
   letter immediately on both sides is part of the word; any other `'` or `-`
   is a separator. So `mother-in-law` is one word and `a--b` is two. The
   apostrophe letters of the 2017 Kazakh Latin alphabet are not supported:
   `tilin'` is `tilin`.
5. **Separators.** Every other character separates words: whitespace, digits,
   `_`, en/em dashes (`–` `—`), prime `′` U+2032, fullwidth `＇` U+FF07 and
   all other punctuation and symbols. A digit inside a run splits it:
   `abc123def` is `abc` and `def`. Numbers alone are never words.
6. **No folding.** `ё` and `е` are different letters. Latin `a` (U+0061) and
   Cyrillic `а` (U+0430) are different letters. Scripts are not checked, so a
   run mixing them, such as `aа`, is one word.
7. **No stemming, no stop words.** `книга` and `книги` are different words;
   `the`, `и`, `және` are counted like any word.
8. **Counting.** Count exact occurrences of each normalized word.
9. **Ordering.** Sort by count, highest first. Break ties by the word in
   ascending Unicode code point order (Python's default `str` comparison).
   English and Kazakh Latin letters (all below U+0400) sort before Cyrillic.
   All Kazakh-specific letters (`ә ғ қ ң ө ұ ү һ і`) sort after every Russian
   letter. A word sorts before a longer word it is a prefix of. This order is
   intentional, even though ties may look unordered to Kazakh readers.
10. **Limit.** Return the first 10 items of the list sorted by rule 9, or all of
    them if there are fewer. Ties at 10th place are therefore resolved by
    rule 9.
11. **No words.** If there are no words (empty string, only whitespace, digits
    or punctuation), return `[]`.
12. **No special cases.** URLs, e-mail addresses and numbers get no special
    handling: `example.com` is `example` and `com`.

## Examples

Inputs and expected outputs are Python literals. The Rule column names the
rule each example pins down.

| #  | Rule | Input | Expected output |
| -- | ---- | ----- | --------------- |
| 1  | 8, 9 | `'cat dog cat bird dog cat'` | `[('cat', 3), ('dog', 2), ('bird', 1)]` |
| 2  | 2.2 | `'The the THE cat'` | `[('the', 3), ('cat', 1)]` |
| 3  | 3 | `'Привет мир привет'` | `[('привет', 2), ('мир', 1)]` |
| 4  | 3 | `'Сәлем әлем сәлем!'` | `[('сәлем', 2), ('әлем', 1)]` |
| 5  | 2.2 | `'ҚАЗАҚСТАН Қазақстан қазақстан'` | `[('қазақстан', 3)]` |
| 6  | 5 | `'hello,world;hello.'` | `[('hello', 2), ('world', 1)]` |
| 7  | 5 | `'abc123def 42 abc'` | `[('abc', 2), ('def', 1)]` |
| 8  | 5 | `'snake_case snake'` | `[('snake', 2), ('case', 1)]` |
| 9  | 4 | `"don't Don't dont"` | `[("don't", 2), ('dont', 1)]` |
| 10 | 2.5 | `"don\u2019t don't"` | `[("don't", 2)]` |
| 11 | 4 | `"'hello' hello dogs' dogs"` | `[('dogs', 2), ('hello', 2)]` |
| 12 | 4 | `'well-known well known'` | `[('known', 1), ('well', 1), ('well-known', 1)]` |
| 13 | 4 | `'кто-то кто-то'` | `[('кто-то', 2)]` |
| 14 | 5 | `'Москва — столица, Москва – город'` | `[('москва', 2), ('город', 1), ('столица', 1)]` |
| 15 | 4, 5 | `'-cat- cat covid-19 covid'` | `[('cat', 2), ('covid', 2)]` |
| 16 | 6 | `'ёлка елка'` | `[('елка', 1), ('ёлка', 1)]` |
| 17 | 6 | `'a \u0430'` | `[('a', 1), ('\u0430', 1)]` |
| 18 | 2.3 | `'\u0438\u0306 \u0439'` | `[('\u0439', 2)]` |
| 19 | 7 | `'книга книги книга'` | `[('книга', 2), ('книги', 1)]` |
| 20 | 7 | `'и и и в в на дом'` | `[('и', 3), ('в', 2), ('дом', 1), ('на', 1)]` |
| 21 | 9 | `'ә я'` | `[('я', 1), ('ә', 1)]` |
| 22 | 9 | `'я a'` | `[('a', 1), ('я', 1)]` |
| 23 | 10 | `'a b c d e f g h i j k'` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` |
| 24 | 10 | `'k k a b c d e f g h i j j'` | `[('j', 2), ('k', 2), ('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1)]` |
| 25 | 5 | `'cat\ncat\tdog'` | `[('cat', 2), ('dog', 1)]` |
| 26 | 3 | `'cat кот мен cat'` | `[('cat', 2), ('кот', 1), ('мен', 1)]` |
| 27 | 12 | `'example.com'` | `[('com', 1), ('example', 1)]` |
| 28 | 11 | `''` | `[]` |
| 29 | 11 | `'   '` | `[]` |
| 30 | 11 | `'123 !!! ---'` | `[]` |
| 31 | 2.4 | `'İstanbul istanbul'` | `[('istanbul', 2)]` |
| 32 | 2.4 | `'за\u0301мок замок'` | `[('замок', 2)]` |
| 33 | 2.4 | `'cafe\u0301 caf\u00e9'` | `[('caf\u00e9', 2)]` |
| 34 | 2.1 | `'инфор\u00adмация информация'` | `[('информация', 2)]` |
| 35 | 2.1 | `'zero\u200bwidth zerowidth'` | `[('zerowidth', 2)]` |
| 36 | 2.5 | `"don\u02bct don't"` | `[("don't", 2)]` |
| 37 | 2.5, 4 | `'\u2018hello\u2019 hello'` | `[('hello', 2)]` |
| 38 | 2.5 | `'кто\u2011то кто\u2010то кто-то'` | `[('кто-то', 3)]` |
| 39 | 6 | `'a\u0430'` | `[('a\u0430', 1)]` |
| 40 | 4 | `"mother-in-law rock'n'roll"` | `[('mother-in-law', 1), ("rock'n'roll", 1)]` |
| 41 | 4 | `"a--b a'-b"` | `[('a', 2), ('b', 2)]` |
| 42 | 4 | `'бала-шаға бала'` | `[('бала', 1), ('бала-шаға', 1)]` |
| 43 | 2.2 | `'Ilim ılım ilim'` | `[('ilim', 2), ('ılım', 1)]` |
| 44 | 2.1 | `'a\u200eb a\u200cb a\u200db a\u2060b \ufeffab'` | `[('ab', 5)]` |
| 45 | 2.4 | `'е\u0300 \u0450 a\u0903b a\u20ddb'` | `[('ab', 2), ('\u0450', 2)]` |
| 46 | 3 | `'\u02c8hello hello'` | `[('hello', 1), ('\u02c8hello', 1)]` |
| 47 | 4 | `"'cat- a-\u0430"` | `[('a-\u0430', 1), ('cat', 1)]` |
| 48 | 2.3 | `'\uff43\uff41\uff54 cat \ufb01nd find'` | `[('cat', 1), ('find', 1), ('\ufb01nd', 1), ('\uff43\uff41\uff54', 1)]` |
| 49 | 3, 5 | `'x\u02bby x\u2032y x\uff07y'` | `[('x', 2), ('y', 2), ('x\u02bby', 1)]` |
| 50 | 2.5, 4 | `'\u2010cat\u2011 cat'` | `[('cat', 2)]` |
| 51 | 4 | `"tilin' tilin"` | `[('tilin', 2)]` |
| 52 | 9, 10 | `'я a b c d e f g h i j'` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` |

Rule 1: `top_words(None)` and `top_words(b'cat')` raise `TypeError`.
With `class S(str): pass`, `top_words(S('cat cat'))` returns `[('cat', 2)]`
and `type(top_words(S('cat'))[0][0]) is str` is true. With
`class L(str): def lower(self): return 'x'`, `top_words(L('Cat cat'))` returns
`[('cat', 2)]`, and with `class T(str): def __iter__(self): return iter('dog')`,
`top_words(T('cat cat'))` returns `[('cat', 2)]`.