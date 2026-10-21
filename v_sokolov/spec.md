# Specification: `top_words`

## Purpose

A news site shows a "Trending words" box beside each article. `top_words` finds the 10 most common words in the article text and how many times each occurs. Supported: English, Russian and Kazakh in Cyrillic script. Latin-script Kazakh is out of scope (an apostrophe used as a letter, as in some Latin Kazakh spellings such as `a'`, would be dropped when trailing but kept when inner, see rule 2).

## Interface

- Language: Python 3.9 or later. File: `top_words.py`. Function: `top_words(text: str) -> list[tuple[str, int]]`.
- `text` must be a `str` or a subclass of `str`. Any other type (`None`, `bytes`, `list`, `int`, `bool`, ...) raises `TypeError`; the message is not specified. A `str` subclass is treated as the plain `str` holding exactly the same code points (as `str.__str__(text)` gives): no method of the subclass (`__str__`, `__iter__`, `__getitem__`, `lower`, ...) is called.
- The function returns a new list of `(word, count)` tuples (`count` is an `int`); each word is an exact `str` and each pair an exact `tuple`. It prints nothing, does not modify its argument and has no other side effects.
- The result depends only on `text` and on the Unicode database of the running Python interpreter (`isalpha`, categories, NFC and `lower()` follow that version; behavior for code points unassigned in it is not specified). Finding the words takes O(`len(text)`); the cost of ordering the distinct words is not specified, but the result must equal the first 10 items of the fully ordered list of rule 7.

## Definitions

- **Letter:** a character `c` with `c.isalpha()` true, *except* U+02BC. This includes Latin, Cyrillic (including Kazakh ә ғ қ ң ө ұ ү һ і) and other modifier letters such as U+02BB. Digits (ASCII and non-ASCII, `²`, `٣`, ...), whitespace, punctuation (including `_`), symbols, format characters (for example the soft hyphen U+00AD and the zero-width joiner U+200D), surrogates and all other code points that are neither letters nor marks are not letters, so they are separators.
- **Mark:** a combining character (Unicode category starting with `M`), for example the stress mark U+0301.
- **Apostrophe:** one of `'` (U+0027), `’` (U+2019), `ʼ` (U+02BC).
- **Hyphen:** only `-` (U+002D). All other dashes (`–`, `—`, the Unicode hyphen U+2010, the non-breaking hyphen U+2011) and other quote-like characters (`` ` ``, `´`, `‘` U+2018) are ordinary separators.
- **Unit:** one letter followed by zero or more marks.

## Rules (applied in this order)

1. **Normalize.** Convert `text` to Unicode NFC (not NFKC).
2. **Find words.** Read left to right, taking the longest match of this grammar: `word = unit+ ( (apostrophe | hyphen) unit+ )*`. So an apostrophe or hyphen belongs to a word only when it is directly between two units. At the start or end of a run, next to another apostrophe or hyphen, or next to a non-letter, it is a separator. A mark directly after an apostrophe or hyphen does not count as a unit start, so that apostrophe or hyphen is a separator too. A mark that does not follow a letter is a separator. All characters not in a word are separators and are skipped.
3. **Normalize apostrophes.** Inside a word each apostrophe (`’`, `ʼ`) is replaced by `'`. (The order of rules 3 and 4 makes no visible difference.)
4. **Case.** Lowercase each whole word with `str.lower()` (so the final-sigma rule of Python applies); the result is returned as is, with no second normalization, so it may contain marks, e.g. `İ` becomes `i` plus U+0307. This is not case folding: `ß` and `ss`, or `ς` and `σ`, stay different. `Cat` and `cat` are the same word.
5. **No other normalization.** No stemming, no stop-word removal, no letter folding: `ё` and `е` are different letters, and Latin `a` (U+0061) and Cyrillic `а` (U+0430) are different letters.
6. **Numbers.** (A corollary of the Definitions and rule 2, stated for clarity.) Every non-letter, including digits and numeric symbols such as `²`, `٣`, `Ⅳ`, is a separator. (Numeric characters that `isalpha()` accepts, such as `三`, are letters.) Digit sequences are never words and a digit splits a word: `3rd` gives `rd`, `covid-19` gives `covid`.
7. **Ordering.** Sort by count, highest first. Words with equal counts are sorted by the word itself (after rules 3 and 4) in ascending Unicode code point order (Python's default `str` comparison). This is *not* the alphabetical order of any language: for example Kazakh `ә` (U+04D9) sorts after `қ` (U+049B).
8. **Length.** Return at most 10 tuples (the first 10 after rule 7). If there are fewer than 10 distinct words, return all of them. If there are no words, return `[]`.
9. **Other scripts.** Text in other scripts is processed by the same rules (unspaced Chinese text is one long word) but this is not a goal.

## Examples

Inputs and outputs are Python literals; characters that are hard to see are written with `\u` escapes.

| # | Input | Output | Decision shown |
|---|-------|--------|----------------|
| 1 | `"cat dog cat bird dog cat"` | `[('cat', 3), ('dog', 2), ('bird', 1)]` | basic counting, order by count |
| 2 | `"Cat cat CAT"` | `[('cat', 3)]` | case-insensitive (rule 4) |
| 3 | `"Кошка кошка КОШКА собака"` | `[('кошка', 3), ('собака', 1)]` | Cyrillic (rules 2, 4) |
| 4 | `"Қазақ қазақ ӘЛІ әлі"` | `[('қазақ', 2), ('әлі', 2)]` | Kazakh letters; tie in code point order, *not* Kazakh alphabet (rule 7) |
| 5 | `"b a c a b"` | `[('a', 2), ('b', 2), ('c', 1)]` | tie broken by word, not by first appearance (rule 7) |
| 6 | `"B a"` | `[('a', 1), ('b', 1)]` | sort key is the lowercased word (rules 4, 7) |
| 7 | `"don't don\u2019t dont"` | `[("don't", 2), ('dont', 1)]` | inner apostrophe, `’` normalized (rules 2, 3) |
| 8 | `"don\u2019t dont"` | `[("don't", 1), ('dont', 1)]` | sorting uses the normalized word: `don't` < `dont` although U+2019 > `t` (rules 3, 7) |
| 9 | `"don\u02bct don't"` | `[("don't", 2)]` | U+02BC is an apostrophe, not a letter (rules 2, 3) |
| 10 | `"well-known well known"` | `[('known', 1), ('well', 1), ('well-known', 1)]` | inner hyphen keeps one word (rule 2) |
| 11 | `"well\u2013known well\u2011known well\u2010known"` | `[('known', 3), ('well', 3)]` | other dashes are separators (Definitions) |
| 12 | `"rock--roll rock''roll"` | `[('rock', 2), ('roll', 2)]` | doubled hyphen or apostrophe splits (rule 2) |
| 13 | `"a-'b a'-b"` | `[('a', 2), ('b', 2)]` | mixed hyphen and apostrophe splits (rule 2) |
| 14 | `"rock'n'roll"` | `[("rock'n'roll", 1)]` | a one-letter run between apostrophes stays in the word (rule 2) |
| 15 | `"'tis word-"` | `[('tis', 1), ('word', 1)]` | leading apostrophe, trailing hyphen dropped (rule 2) |
| 16 | `"\u02bctis \u2018tis dog\u2019"` | `[('tis', 2), ('dog', 1)]` | edge apostrophes dropped; U+2018 is a separator (rule 2, Definitions) |
| 17 | `"-hello- 'it' covid-19"` | `[('covid', 1), ('hello', 1), ('it', 1)]` | edge characters dropped, digits separate (rules 2, 6) |
| 18 | `"room 101 101 room"` | `[('room', 2)]` | numbers ignored (rule 6) |
| 19 | `"b2b 3rd"` | `[('b', 2), ('rd', 1)]` | a digit splits a word (rule 6) |
| 20 | `"x\u00b2 \u0663y"` | `[('x', 1), ('y', 1)]` | non-ASCII numeric characters are separators (rule 6) |
| 21 | `"snake_case"` | `[('case', 1), ('snake', 1)]` | underscore is a separator (Definitions) |
| 22 | `"co\u00adoperate"` | `[('co', 1), ('operate', 1)]` | a soft hyphen (format character) is a separator, by design (Definitions) |
| 23 | `"a\u200db"` | `[('a', 1), ('b', 1)]` | a zero-width joiner is a separator, by design (Definitions) |
| 24 | `"a\ud800b"` | `[('a', 1), ('b', 1)]` | a lone surrogate is a separator (Definitions) |
| 25 | `"\u4e00\u4e8c\u4e09 x"` | `[('x', 1), ('\u4e00\u4e8c\u4e09', 1)]` | numeric ideographs are letters (`isalpha()`), so they form a word (rule 6) |
| 26 | `"Hello, world! Hello."` | `[('hello', 2), ('world', 1)]` | punctuation separates words (rule 2) |
| 27 | `"\u0438\u0306 \u0439"` | `[('\u0439', 2)]` | decomposed й equals precomposed й (rule 1) |
| 28 | `"a\u0323\u0301 a\u0301\u0323"` | `[('\u1ea1\u0301', 2)]` | NFC also reorders marks: both spellings are the same word (rule 1) |
| 29 | `"\u043c\u0430\u0301\u043c\u0430 \u043c\u0430\u043c\u0430"` | `[('\u043c\u0430\u0301\u043c\u0430', 1), ('\u043c\u0430\u043c\u0430', 1)]` | stress mark U+0301 stays in the word, so `ма́ма` and `мама` differ (rule 2) |
| 30 | `"\u0301cat"` | `[('cat', 1)]` | a mark that does not follow a letter is a separator (rule 2) |
| 31 | `"a-\u0301b a b"` | `[('a', 2), ('b', 2)]` | a mark after a hyphen: the hyphen is no longer between two units, so the word splits (rule 2) |
| 32 | `"a'\u0301b"` | `[('a', 1), ('b', 1)]` | a mark after an apostrophe: the apostrophe is not between two units, so the word splits (rule 2) |
| 33 | `"\u043c\u0430\u0301-\u043c\u0430"` | `[('\u043c\u0430\u0301-\u043c\u0430', 1)]` | a mark before a hyphen keeps the word whole (rule 2) |
| 34 | `"\u043c\u0430\u0301'\u043c\u0430"` | `[("\u043c\u0430\u0301'\u043c\u0430", 1)]` | the same with an apostrophe: one word, the mark belongs to the unit before it (rule 2) |
| 35 | `"\u0130stanbul"` | `[('i\u0307stanbul', 1)]` | `lower()` result returned unchanged (rule 4) |
| 36 | `"\u0130\u0316 i\u0316\u0307"` | `[('i\u0307\u0316', 1), ('i\u0316\u0307', 1)]` | no NFC after `lower()`: `i\u0307\u0316` and `i\u0316\u0307` stay different words (rule 4) |
| 37 | `"\u00df ss"` | `[('ss', 1), ('\u00df', 1)]` | `ß` and `ss` stay different words (rule 4) |
| 38 | `"\u02bbokina"` | `[('\u02bbokina', 1)]` | U+02BB (okina) is a letter, unlike U+02BC (Definitions) |
| 39 | `"\u039f\u0394\u039f\u03a3 \u03bf\u03b4\u03bf\u03c2 \u03bf\u03b4\u03bf\u03c3"` | `[('\u03bf\u03b4\u03bf\u03c2', 2), ('\u03bf\u03b4\u03bf\u03c3', 1)]` | `lower()` is applied to the whole word, so a final `Σ` becomes `ς`; `ς` and `σ` stay different (rule 4) |
| 40 | `"\u039f\u0394\u039f\u03a3-\u0391\u0392 \u039f\u0394\u039f\u03a3'\u0391\u0392 \u039f\u0394\u039f\u03a3\u2018\u0391\u0392"` | `[('\u03b1\u03b2', 1), ('\u03bf\u03b4\u03bf\u03c2', 1), ('\u03bf\u03b4\u03bf\u03c2-\u03b1\u03b2', 1), ("\u03bf\u03b4\u03bf\u03c3'\u03b1\u03b2", 1)]` | `lower()` sees the whole word: per CPython's final-sigma rule (an apostrophe is case-ignorable, a hyphen is not) `Σ` before a hyphen is final (`ς`) but before an inner apostrophe it is `σ`; with U+2018 the word ends at `Σ` (rule 4) |
| 41 | `"\u0451\u043b\u043a\u0430 \u0435\u043b\u043a\u0430"` | `[('\u0435\u043b\u043a\u0430', 1), ('\u0451\u043b\u043a\u0430', 1)]` | no ё/е folding (rule 5) |
| 42 | `"cat c\u0430t"` | `[('cat', 1), ('c\u0430t', 1)]` | Latin a and Cyrillic а (U+0430) differ (rule 5) |
| 43 | `"\ufb01sh fish"` | `[('fish', 1), ('\ufb01sh', 1)]` | NFC, not NFKC: the ligature U+FB01 stays (rules 1, 5) |
| 44 | `"a b b c c d d e e f f g g h h i i j j k k"` | `[('b', 2), ('c', 2), ('d', 2), ('e', 2), ('f', 2), ('g', 2), ('h', 2), ('i', 2), ('j', 2), ('k', 2)]` | cut at 10 drops the lower count even though `a` sorts first (rules 7, 8) |
| 45 | `"a b c d e f g h i j k l"` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | limit of 10, ties by word (rules 7, 8) |
| 46 | `"\u65e5\u672c\u8a9e"` | `[('\u65e5\u672c\u8a9e', 1)]` | other scripts: one long word (rule 9) |
| 47 | `""` | `[]` | empty input (rule 8) |
| 48 | `"123 !!! ---"` | `[]` | no words (rule 8) |
| 49 | `"'' - ' --"` | `[]` | only apostrophes and hyphens (rule 2) |
| 50 | `42`, `None`, `b"cat"`, `["cat"]`, `True` | raises `TypeError` | invalid type (Interface) |
| 51 | `class S(str): __str__ = lambda s: "X"; __iter__ = lambda s: iter("zzz"); lower = lambda s: "X"`, then `S("Cat")` | `[('cat', 1)]` | a `str` subclass is processed as a plain `str`; overridden methods are ignored; the returned words and tuples are exactly `str` and `tuple` (Interface) |
