# Top words specification

A news site shows a "Trending words" box beside each article. `top_words` finds the 10 most common words of an article and how often each appears. Articles may be in English, Russian or Kazakh (Cyrillic or Latin letters; no language detection is done). The Kazakh Latin alphabet targeted is the 2021 draft (with `ı ä ö ü ğ ş ń ý İ`); the superseded 2017 version, which writes letters as an apostrophe digraph (`s'` = ш), is not supported.

## Interface

- Python 3, file `top_words.py`, function `top_words(text: str) -> list[tuple[str, int]]`.
- `text` is **plain text**: HTML, Markdown or other markup is not interpreted, and its characters are treated like any other text (the caller strips markup first).
- Returns at most 10 `(word, count)` tuples, ordered as described in "Output order".
- If `text` is not an instance of `str` (for example `None` or `bytes`), raise `TypeError`.
- Performance: the running time must be linear in the length of `text` (plus sorting the distinct words), so that articles of up to 10 MB are processed in seconds. A quadratic implementation (for example repeated string concatenation or `list.count` per word) does not conform.

## Processing steps

Apply these steps in this order:

1. **Clean and normalize** `text`: remove every soft hyphen (U+00AD) and every variation selector (U+FE00 to U+FE0F and U+E0100 to U+E01EF), then normalize to Unicode NFC (`unicodedata.normalize("NFC", text)`). Soft hyphens are invisible hyphenation hints inserted by content systems, so `co\u00ADoperation` is the single word `cooperation`.
2. **Tokenize** into words (next section).
3. **Normalize connectors** inside each word: replace U+2019 and U+02BC with U+0027, and U+2010 and U+2011 with U+002D.
4. **Case-fold** each word: first replace every U+0130 (`İ`) with `i`, then apply `str.casefold()`, then normalize the result to NFC again. Words are compared and returned in this form. (`casefold` can change a word: `ß` becomes `ss`, and it can decompose characters such as `ΐ` (U+0390), which the final NFC puts back together.)
5. **Count** identical words and **sort** them.

## Definition of a word

- A *letter* is any character whose Unicode general category starts with `L`, **except** U+02BC, which is an apostrophe (below). Letters of any script count, including Latin, Cyrillic and the Kazakh letters ә ғ қ ң ө ұ ү һ і.
- A *mark* is any character whose general category starts with `M` (combining mark).
- A *connector* is an apostrophe (U+0027, U+2019 or U+02BC) or a hyphen (U+002D hyphen-minus, U+2010 hyphen or U+2011 non-breaking hyphen). No other character is a connector. In particular U+2018, U+00B4, the backtick, the en/em dashes and the minus sign are *not* connectors.
- A *word* is a maximal sequence that **starts with a letter**, in which every letter may be followed by any number of marks, and where one letter-with-marks may be joined to the next by exactly **one** connector. In other words, a connector is part of the word only when it has a letter or mark immediately before it **and a letter immediately after it**. So `rock'n'roll`, `well-known` and `don't-do` are single words, while `'dog'`, `dogs'`, `-dog`, `well--known`, `a'-b` and `a-\u0301` are not: there the connector is a separator.
- A mark that does not follow a letter or another mark of the same word (at the start of the text, after a separator, or right after a connector) is a separator. A sequence of marks alone is therefore never a word.
- Every other character is a **separator**: whitespace (including newlines and tabs), punctuation, symbols, em/en dashes, underscore, and all digits. Digits are never part of a word and numbers are never counted: `b2b` gives `b` and `b`; `2024` gives nothing.
- No stemming, lemmatization, script splitting or unification: `привет-world` is one word; Latin `a` (U+0061) and Cyrillic `а` (U+0430) are different letters; `ё` and `е` are different letters.
- Scripts written without spaces (for example Chinese) get no special handling: a run of letters is one word.
- No stop words are removed: `the`, `и` and `және` are counted like any other word.

## Output order and display form

- Sort by count descending; ties are broken by the word ascending in Unicode code point order (Python's default string comparison; this is *not* a language-specific alphabetical order, so for example Kazakh `ә` sorts after every basic Russian letter). A word that is a prefix of another sorts first (`well` before `well-known`).
- Return the first 10 items of that order. If there are fewer than 10 distinct words, return all of them. If there are none, return `[]`.
- The result is a new `list` of `tuple`s `(str, int)`.
- Words are returned in their case-folded form, which is what the "Trending words" box shows: proper nouns are lowercase (`москва`, `astana`) and `Straße` is shown as `strasse`. The original spelling is not kept; choosing a display form is the caller's business.

## Known limitations

- Case-folding is locale-independent. The only language-specific exception is `İ` → `i` (needed for the Kazakh Latin alphabet); Turkish dotless `ı` (U+0131) stays a different letter from `i`.
- Removing U+00AD and variation selectors changes words only by dropping invisible characters.

## Examples

Each row exercises a decision above: for most rows, another plausible reading of the original problem would give a different output (the first rows are plain sanity checks). `\uXXXX` in an input is a single character with that code point.

| Decision | Input | Expected output |
| --- | --- | --- |
| Basic counting | `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| Case-insensitive (Latin) | `"Cat CAT cat"` | `[("cat", 3)]` |
| Case-insensitive (Cyrillic) | `"Привет привет ПРИВЕТ"` | `[("привет", 3)]` |
| Kazakh letters | `"Қазақстан қазақстан Әлем"` | `[("қазақстан", 2), ("әлем", 1)]` |
| ё and е are different; ties by code point (е U+0435 < ё U+0451) | `"еж ёж"` | `[("еж", 1), ("ёж", 1)]` |
| Punctuation separates | `"hi, hi! hi? (hi)"` | `[("hi", 4)]` |
| Whitespace of any kind separates | `"cat\ncat\tdog"` | `[("cat", 2), ("dog", 1)]` |
| Apostrophes U+0027, U+2019, U+02BC are one word, written as U+0027 | `"don't don\u2019t don\u02BCt"` | `[("don't", 3)]` |
| Edge apostrophes are separators | `"'dog' dog"` | `[("dog", 2)]` |
| Trailing apostrophe (plural possessive) is a separator | `"dogs' dogs"` | `[("dogs", 2)]` |
| Leading hyphen is a separator | `"-dog dog"` | `[("dog", 2)]` |
| Chained apostrophes | `"rock'n'roll"` | `[("rock'n'roll", 1)]` |
| Internal hyphen joins; ties by code point (prefix sorts first) | `"well-known well known"` | `[("known", 1), ("well", 1), ("well-known", 1)]` |
| Doubled hyphen separates | `"well--known"` | `[("known", 1), ("well", 1)]` |
| Apostrophe and hyphen together in one word | `"don't-do"` | `[("don't-do", 1)]` |
| Apostrophe next to hyphen separates | `"a'-b"` | `[("a", 1), ("b", 1)]` |
| Em dash is a separator | `"a \u2014 b"` | `[("a", 1), ("b", 1)]` |
| U+2010 and U+2011 are hyphen connectors, written as U+002D | `"a\u2010b a\u2011b a-b"` | `[("a-b", 3)]` |
| Connector next to a digit is an edge | `"a-1 3-d"` | `[("a", 1), ("d", 1)]` |
| Digits are separators | `"2024 abc 123abc"` | `[("abc", 2)]` |
| Letters and digits mixed | `"b2b"` | `[("b", 2)]` |
| Underscore separates | `"foo_bar"` | `[("bar", 1), ("foo", 1)]` |
| No stop words removed | `"the the a"` | `[("the", 2), ("a", 1)]` |
| Tie-break by code point | `"b a c b a"` | `[("a", 2), ("b", 2), ("c", 1)]` |
| Tie-break by code point, not alphabet (Kazakh ә U+04D9 after я U+044F) | `"\u04D9 \u044F"` | `[("\u044F", 1), ("\u04D9", 1)]` |
| casefold, not lower (ß → ss) | `"Straße STRASSE"` | `[("strasse", 2)]` |
| İ (U+0130) is mapped to i (Kazakh Latin) | `"\u0130nternet internet"` | `[("internet", 2)]` |
| casefold result is re-normalized (ΐ U+0390 folds to a decomposed form) | `"\u0390"` | `[("\u0390", 1)]` |
| No script splitting | `"привет-world приветworld"` | `[("привет-world", 1), ("приветworld", 1)]` |
| Latin a (U+0061) and Cyrillic а (U+0430) are different | `"a \u0430"` | `[("a", 1), ("\u0430", 1)]` |
| Latin i (U+0069) and Kazakh і (U+0456) are different | `"i \u0406"` | `[("i", 1), ("\u0456", 1)]` |
| NFC normalization (и + U+0306 vs й U+0439) | `"и\u0306 й"` | `[("й", 2)]` |
| Combining marks only is not a word | `"\u0301"` | `[]` |
| A mark at the start of a word is dropped | `" \u0301abc"` | `[("abc", 1)]` |
| A mark right after a connector does not make the connector part of a word | `"a-\u0301"` | `[("a", 1)]` |
| Ten words, all ties: first ten alphabetically | `"a b c d e f g h i j k"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` |
| Cut-off after ten with a higher count first | `"z z a b c d e f g h i j"` | `[("z", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` |
| Soft hyphen U+00AD is removed, not a separator | `"co\u00ADoperate cooperate"` | `[("cooperate", 2)]` |
| Variation selector U+FE0F is removed | `"a\uFE0F a"` | `[("a", 2)]` |
| U+2018 is a separator; U+2019 at the edge too | `"\u2018dog\u2019"` | `[("dog", 1)]` |
| Scripts without spaces: a run of letters is one word | `"中文"` | `[("中文", 1)]` |
| Markup is not interpreted (plain text) | `"<b>hi</b>"` | `[("b", 2), ("hi", 1)]` |
| Empty text | `""` | `[]` |
| No words | `"123 !!!"` | `[]` |
| Not a str: None | `None` | raises `TypeError` |
| Not a str: bytes | `b'abc'` | raises `TypeError` |