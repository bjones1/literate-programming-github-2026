# Top words: specification

Source: [Top words](https://github.com/bjones1/literate-programming-book/blob/main/exercises/top-words.md).

## Interface

Write `top_words(text)` in `top_words.py`, using only the standard library. The target is Python 3.12, so character categories, case folding, and NFC follow its `unicodedata` tables (Unicode 15.0). Newer Pythons may classify characters added after Unicode 15.0 differently. All examples below give the same results on Python 3.12, 3.13, and 3.14.

- `text` is the article's plain text as a `str` (a subclass of `str` is accepted: first convert it with `str.__str__(text)` to the plain `str` with the same code points, so no overridden method is used). It is never treated as a file name, URL, or HTML. Any other type raises `TypeError`.
- The result is a new `list` of at most 10 `(word, count)` tuples, where `word` is a `str` and `count` is a positive `int`.
- The function does no I/O, keeps no state between calls, and does not modify its argument.

## Normalization

Before splitting into words, convert `text` to a plain `str` and delete every soft hyphen (U+00AD). Deletion happens before normalization, so a soft hyphen never blocks composition. Then normalize the whole text with
`unicodedata.normalize("NFC", unicodedata.normalize("NFD", t).casefold())`, where `t` is the text after deleting soft hyphens.

Deleting soft hyphens joins words that web pages break invisibly (`co\u00adoperate` is `cooperate`). Normalization makes counting case-insensitive under Unicode default (non-Turkic) case folding (`Straße` and `STRASSE` match) and makes canonically equivalent spellings match (precomposed `й` and `и` + U+0306 match). Returned words are in this normalized form (with rule 5 below applied). No other change is made: no transliteration, accent removal, stemming, stop-word removal, or language-specific case rules. `е`/`ё`, `и`/`й`, `а`/`ә`, and Latin `a`/Cyrillic `а` stay different words.

## Words

Character classes use Unicode general categories from Python's `unicodedata.category`:

- A **letter** has a category starting with `L`. This includes U+02BC (ʼ, `Lm`).
- A **mark** has a category starting with `M` (combining accents and similar).
- A **joiner** is U+0027 (`'`), U+2019 (`’`), or U+002D (`-`).
- Every other character, including digits, whitespace, other punctuation, other dashes such as U+2010 and U+2013, symbols, and most emoji, is a **separator**. The category decides, not the emoji property: ℹ (U+2139) is `Ll`, a letter, and a following U+FE0F is a mark.

Scan the normalized text from left to right. At each position, the longest match of this pattern is a word. After a word, scanning continues at the first character after it. If no word starts at a position, that character is skipped. The pattern is the definition. The numbered rules below only explain it.

```text
word = letter (letter | mark)* ( joiner letter (letter | mark)* )*
```

In words:

1. A word starts with a letter. It continues through letters and marks.
2. A joiner stays inside the word only when it comes right after a letter or mark of the word and right before a letter. So `don't`, `rock'n'roll`, `кое-что`, and `Қазақстан-Ресей` are each one word. This keeps English contractions and Russian and Kazakh hyphenated words whole.
3. Otherwise a joiner is a separator. Leading, trailing, and doubled joiners never belong to a word, so `'cat'` is `cat` and `well--known` is `well` and `known`.
4. A mark that is not part of a word acts as a separator. A joiner followed by a mark is not followed by a letter, so `a'\u0301b` is `a` and `b`.
5. In each word, every U+2019 is replaced by U+0027, so `don’t` and `don't` are the same word. U+02BC (ʼ) is a letter, not a joiner, so it is kept as it is: `donʼt` is a different word from `don't`.
6. Case folding can produce a mark. For example, `İ` folds to `i` + U+0307, which is a different word from `i`. This is the standard Unicode rule; there is no Turkish or Kazakh special case, so in Kazakh Latin text `İLİM` and `ilim` are different words, and `I` folds to `i`, not `ı`.
7. Letters from different scripts with no separator between them form one word. Digits are separators, so `covid19` gives `covid`.
8. Single-letter words and stop words (`the`, `и`, `және`) count like any other word.

## Counting and order

Count how many times each distinct word occurs. Sort by count, highest first. Break ties by ascending Python string order (Unicode code points, not locale or alphabet order). Return the first 10 entries. Do not add extra entries when the 10th and 11th words have the same count. Return all entries when there are fewer than 10 distinct words, and `[]` when there are none.

## Examples

Inputs and outputs use Python literal syntax. Each output is the complete return value, or the exception raised. Several inputs in one cell are separate calls.

| ID | Decision | Input | Expected output |
| --- | --- | --- | --- |
| E1 | Count and sort by count | `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| E2 | Case folding in English, Russian, Kazakh | `"Cat CAT cat"`<br>`"Мир МИР мир"`<br>`"Қазақ ҚАЗАҚ"` | `[("cat", 3)]`<br>`[("мир", 3)]`<br>`[("қазақ", 2)]` |
| E3 | Full case folding, not `lower()` | `"Straße STRASSE strasse"` | `[("strasse", 3)]` |
| E4 | Canonical equivalents match | `"\u0439 \u0438\u0306 \u0419"`<br>`"caf\u00e9 cafe\u0301"` | `[("\u0439", 3)]`<br>`[("caf\u00e9", 2)]` |
| E5 | Similar letters stay different | `"ёлка елка ёлка"`<br>`"a а ә"` (Latin, Cyrillic, Kazakh) | `[("ёлка", 2), ("елка", 1)]`<br>`[("a", 1), ("а", 1), ("ә", 1)]` |
| E6 | Inner apostrophes join; curly becomes straight | `"don't don’t DON'T"`<br>`"rock'n'roll"` | `[("don't", 3)]`<br>`[("rock'n'roll", 1)]` |
| E7 | U+02BC is a letter, not an apostrophe | `"don\u02bct don't"` | `[("don't", 1), ("don\u02bct", 1)]` |
| E8 | Inner hyphens join | `"кое-что из-за кое-что"`<br>`"Қазақстан-Ресей"` | `[("кое-что", 2), ("из-за", 1)]`<br>`[("қазақстан-ресей", 1)]` |
| E9 | Edge and doubled joiners split; a joiner before a mark splits | `"'cat' -dog- well--known"`<br>`"a'\u0301b"` | `[("cat", 1), ("dog", 1), ("known", 1), ("well", 1)]`<br>`[("a", 1), ("b", 1)]` |
| E10 | Other dashes, punctuation, and most emoji split; ℹ is a letter | `"well\u2013known cat,dog cat🙂dog"`<br>`"\u2139\ufe0f info"`<br>`"из\u2011за из\u2010за"` | `[("cat", 2), ("dog", 2), ("known", 1), ("well", 1)]`<br>`[("info", 1), ("\u2139\ufe0f", 1)]`<br>`[("за", 2), ("из", 2)]` |
| E11 | Digits split; numbers are not words | `"COVID-19 covid19 2026"` | `[("covid", 2)]` |
| E12 | Soft hyphens are deleted before normalization | `"co\u00adoperate cooperate"`<br>`"cafe\u00ad\u0301 caf\u00e9"` | `[("cooperate", 2)]`<br>`[("caf\u00e9", 2)]` |
| E13 | Folding `İ` keeps the dot as a mark; no Turkic special case | `"\u0130 I i"`<br>`"\u0130L\u0130M ilim"` | `[("i", 2), ("i\u0307", 1)]`<br>`[("ilim", 1), ("i\u0307li\u0307m", 1)]` |
| E14 | Mixed scripts without a separator are one word | `"catкот catкот кот"` | `[("catкот", 2), ("кот", 1)]` |
| E15 | Marks inside a word stay; stray marks are separators | `"по\u0301ле по\u0301ле"`<br>`"\u0301cat \u0301 cat"` | `[("по\u0301ле", 2)]`<br>`[("cat", 2)]` |
| E16 | Stop words and one-letter words count | `"и в и the a the және"` | `[("the", 2), ("и", 2), ("a", 1), ("в", 1), ("және", 1)]` |
| E17 | Ties use code-point order | `"zebra apple кот zebra apple кот"`<br>`"bb b"` | `[("apple", 2), ("zebra", 2), ("кот", 2)]`<br>`[("b", 1), ("bb", 1)]` |
| E18 | Cut at 10, even inside a tie | `"k j i h g f e d c b a"`<br>`"a b c d e f g h i j k k"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]`<br>`[("k", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` |
| E19 | No words gives an empty list | `""`<br>`"123 -- !!! 🙂"` | `[]`<br>`[]` |
| E20 | Non-strings are rejected | `None`<br>`b"cat"`<br>`["cat"]` | `TypeError`<br>`TypeError`<br>`TypeError` |
