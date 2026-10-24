# Top words: specification

Author: b.yunussov

> A news site shows a "Trending words" box beside each article. Write a Python
> function that finds the 10 most common words in an article's text, along with
> how many times each one appears. Articles may be in English, Russian, or
> Kazakh.

Sections 1 and 2 are normative; the prose and tables after them add no rules
and only illustrate them. Input and output cells are Python literals (or
`TypeError` for a raised exception); `\u` escapes show invisible or look-alike
characters, or characters whose exact code point matters.

## 1. Interface

- `top_words(text)` in `top_words.py`; Python 3.10+, standard library only,
  deterministic, no side effects.
- Raises `TypeError` unless `isinstance(text, str)`; never raises for a `str`.
- Returns a `list` of at most 10 `(word, count)` tuples of `(str, int)`, or
  `[]` if the text has no words. The box shows each word as section 2, step 4
  produces it; recovering the original spelling is out of scope.
- "Kazakh" means Cyrillic script. In Latin-script Kazakh, words with `I`,
  `ı` or `İ` may be miscounted, since Turkic case rules are not applied.
- Not handled specially (section 2 still defines the result): language
  detection, transliteration, HTML, and scripts without spaces between words.
- Character properties come from the interpreter's Unicode database
  (`unicodedata.unidata_version`).

## 2. Procedure

A **letter** has `c.isalpha()`, a **digit** has `c.isdecimal()`, and a
**mark** has a `unicodedata.category(c)` starting with `M`. A **joiner** is
`'` (U+0027), `’` (U+2019), `-` (U+002D), `‐` (U+2010), or `‑` (U+2011).
Every other character left after step 1 is a separator.

1. Delete every character of category `Cf` (soft hyphen, word joiner, BOM, ...)
   except U+200B zero-width space, which is a separator (UAX #29 also breaks
   there). Also delete the Default_Ignorable_Code_Point characters assigned
   outside `Cf` as of Unicode 16.0: U+034F, U+115F, U+1160, U+17B4, U+17B5,
   U+180B–U+180D, U+180F, U+3164, U+FFA0, U+FE00–U+FE0F, and U+E0100–U+E01EF.
2. Scan left to right. A token is the longest match of
   `(A M*)+ (J (A M*)+)*` (spaces for readability), where `A` is a letter or
   digit, `M` a mark and `J` a joiner. Everything outside tokens is discarded.
3. Discard tokens that contain no letter.
4. The word is `NFC(NFD(token).casefold())`, with `’` then replaced by `'`
   and U+2010, U+2011 by `-`.
5. Count words; sort by count, highest first, then by word in code point order
   (Python `str` comparison). Return the first 10.

## 3. Words

| Input | Expected output | Decision |
| --- | --- | --- |
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | list of `(word, count)` |
| `""` | `[]` | no words |
| `None` | `TypeError` | only `str` |
| `b"cat"` | `TypeError` | bytes are not decoded |
| `"Hello, world! Hello?"` | `[("hello", 2), ("world", 1)]` | punctuation separates |
| `"«Астана» — бас қала."` | `[("астана", 1), ("бас", 1), ("қала", 1)]` | any script, any punctuation |
| `"2024 — 15% 3.14"` | `[]` | numbers alone are not words |
| `"COVID-19 5G 1990-х snake_case"` | `[("1990-х", 1), ("5g", 1), ("case", 1), ("covid-19", 1), ("snake", 1)]` | digits join letters; `_` separates |
| `"\uff15G \u0663"` | `[("\uff15g", 1)]` | any `isdecimal()` digit, not only `0`–`9` |
| `"x\u00b2 \u00bdx XXI"` | `[("x", 2), ("xxi", 1)]` | `²`, `½` are not digits |
| `"cat category cats"` | `[("cat", 1), ("category", 1), ("cats", 1)]` | whole words only |
| `"don't don\u2019t 'cat' dogs'"` | `[("don't", 2), ("cat", 1), ("dogs", 1)]` | `'`, `’` join only inside a word |
| `"кто\u0301-то"` | `[("кто\u0301-то", 1)]` | a mark may come before a joiner |
| `"Ростов-на-Дону Нұр-Сұлтан Нұр\u2010Сұлтан Нұр\u2011Сұлтан"` | `[("нұр-сұлтан", 3), ("ростов-на-дону", 1)]` | hyphens join inside a word; U+2010, U+2011 become `-` |
| `"a--b a-\u0301b well-\nknown x\u2212y e\u2014mail"` | `[("a", 2), ("b", 2), ("e", 1), ("known", 1), ("mail", 1), ("well", 1), ("x", 1), ("y", 1)]` | a joiner needs a letter or digit after it; `−`, `—` separate |
| `"don\u2018t don\u02bct"` | `[("don", 1), ("don\u02bct", 1), ("t", 1)]` | `‘` separates; `ʼ` is a letter, so `donʼt` is not `don't` |
| `"news\u00adpaper news\u2060paper news\u200dpaper cat\u200bdog"` | `[("newspaper", 3), ("cat", 1), ("dog", 1)]` | `Cf` is deleted, except U+200B separates |
| `"cat cat\ufe0f cat\u034f \u3164"` | `[("cat", 3)]` | other invisible characters are deleted |
| `"в\u00a0Астане 5\u202f000 км"` | `[("астане", 1), ("в", 1), ("км", 1)]` | no-break spaces separate |
| `"т.\u0435. и т.д. U.S."` | `[("т", 2), ("s", 1), ("u", 1), ("д", 1), ("\u0435", 1), ("и", 1)]` | periods split abbreviations |
| `"за\u0301мок замок \u0345x"` | `[("x", 1), ("за\u0301мок", 1), ("замок", 1)]` | marks stay in a word; a leading mark is dropped, not folded into `ι` |
| `"a\ud800b"` | `[("a", 1), ("b", 1)]` | a lone surrogate separates; no error |
| `"<p>cat</p> &amp;"` | `[("p", 2), ("amp", 1), ("cat", 1)]` | markup is plain text |

## 4. Case and normalization

Folding is `str.casefold()`: not `lower()`, not locale-aware, and applied to
each token, not to the whole text.

| Input | Expected output | Decision |
| --- | --- | --- |
| `"Cat CAT cat Москва МОСКВА"` | `[("cat", 3), ("москва", 2)]` | case ignored; folded word shown |
| `"Қазақ ҚАЗАҚ Әлем әлем"` | `[("қазақ", 2), ("әлем", 2)]` | Kazakh letters fold |
| `"Straße STRASSE \ufb01nance finance"` | `[("finance", 2), ("strasse", 2)]` | full folding, which also expands `ﬁ` |
| `"\u0130stanbul istanbul"` | `[("istanbul", 1), ("i\u0307stanbul", 1)]` | `İ` folds to `i` + U+0307, not `i` |
| `"\u0439 \u0438\u0306"` | `[("\u0439", 2)]` | NFC: `й` equals `и` + breve |
| `"\u1fb3\u0315 \u03b1\u0315\u03b9"` | `[("\u03b1\u0315\u03b9", 2)]` | fold after NFD (canonical caseless match) |
| `"\uff43\uff41\uff54 cat"` | `[("cat", 1), ("\uff43\uff41\uff54", 1)]` | no NFKC |

## 5. No other merging

There is no stemming, no stopword list, and no minimum length. `ё` is not `е`,
and Latin `i` is not Kazakh `і` (U+0456). Words like `the`, `и` and `мен` will
almost always lead the box. This is deliberate: the task asks for the most
common words, and any stopword list would be a guess.

| Input | Expected output | Decision |
| --- | --- | --- |
| `"ещ\u0451 еще б\u0069р б\u0456р"` | `[("б\u0069р", 1), ("б\u0456р", 1), ("еще", 1), ("ещ\u0451", 1)]` | no letter merging |
| `"run runs Tokayev's Tokayev Алматы Алматыда"` | `[("run", 1), ("runs", 1), ("tokayev", 1), ("tokayev's", 1), ("алматы", 1), ("алматыда", 1)]` | no stemming; `'s` is kept |
| `"the cat and the dog и в и"` | `[("the", 2), ("и", 2), ("and", 1), ("cat", 1), ("dog", 1), ("в", 1)]` | no stopwords or minimum length |

## 6. Ranking

| Input | Expected output | Decision |
| --- | --- | --- |
| `"b a b a c"` | `[("a", 2), ("b", 2), ("c", 1)]` | ties by code point, not first appearance |
| `"\u0451ж яма \u04d9 \u0430 apple"` | `[("apple", 1), ("\u0430", 1), ("яма", 1), ("\u0451ж", 1), ("\u04d9", 1)]` | not alphabetical: Latin, then `а`–`я`, then `ё`, `ә` |
| `"k j i h g f e d c b a"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` | ties cut at 10 |
| `"a b c d e f g h i j k k"` | `[("k", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` | count before code point |
