# Top words: specification

`top_words` finds the 10 most common words in a news article, and how many
times each appears, for a "Trending words" box. Articles may be in English,
Russian or Kazakh (Cyrillic, or the 2021 Latin alphabet); there is no
language detection.

## Interface

Define `top_words(text: str) -> list[tuple[str, int]]` in `top_words.py`. It
must run on Python 3.11 and later and use only the standard library.

- `text`: the article as plain text. Any `str` is accepted; a subclass is first
  converted with `str.__str__(text)`. Markup such as HTML is not parsed. Any
  other type raises `TypeError`.
- Returns: a new list of at most 10 `(word, count)` tuples: a word produced by
  the steps below and the number of times it occurs. The function has no side
  effects.

All Unicode data (categories, case folding, normalization) comes from the
running Python; the examples use no character added after Unicode 14.0.

## Steps

A *letter* is a character whose general category starts with `L`; a *digit* is
a character in category `Nd`.

1. **Normalize:** `t = NFC(s.casefold())`, where `s` is the plain `str` from
   the Interface and `NFC` is `unicodedata.normalize("NFC", ...)`.
   Casing is not Turkic (English needs `In` → `in`), so a Kazakh-Latin capital
   `I`, in a headline or at the start of a sentence, folds to `i`, not `ı`:
   `BILIM` gives `bilim`, not `bılım`.
2. **Delete** every character of `t` whose general category starts with `M`
   or is `Cf`, except U+200B ZERO WIDTH SPACE.
3. **Unify joiners** (`'` and `-`): replace U+2018 `‘`, U+2019 `’` and U+02BC
   `ʼ` with U+0027 `'`, and U+2010 `‐` and U+2011 `‑` with U+002D `-`.
4. **Drop stray endings.** An *ending* is a maximal run of letters that is
   all Cyrillic (U+0400 to U+052F) or at most two letters long. First delete
   each ending that comes right after a digit. Then replace with a space each
   ending that comes right after either
   - a `-` whose preceding character is not a letter, `-`, U+200B or
     whitespace (`str.isspace`), or
   - a `'` whose preceding character is a digit.

   A joiner at the start of the text has no preceding character, so it never
   qualifies. Endings joined to letters (`АҚШ-тың`) stay, and a Latin run of
   three or more letters is never an ending, so Kazakh written in Latin script
   keeps such suffixes as words (`«QTJ»-men` counts `men`).
5. **Split:** a word is a maximal run of letters, where a `'` or `-` with a
   letter immediately before and after it also counts as part of the run.
   Every other character separates words.
6. **Count** each distinct word. Do not remove stop words, stem, or
   transliterate.
7. **Sort** by count, highest first; break ties with Python's `<` on `str`
   (code-point order).
8. **Return** the first 10 pairs.

## Examples

Each input is a Python expression passed as `text`, unless the cell shows the
calls; `\uXXXX` is one character. The last column is a plausible reading that
the example rules out.

| Input | Expected output | Rules out |
| --- | --- | --- |
| `"bird cat dog cat dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | sorting alphabetically or by first appearance instead of by count |
| `"Straße STRASSE ҚАЗАҚ қазақ"` | `[("strasse", 2), ("қазақ", 2)]` | case-sensitive counts; ASCII-only lowercasing; `str.lower()`, which keeps `straße` |
| `"б\u0069р\u0069нш\u0069 б\u0456р\u0456нш\u0456 ёж еж"` | `[("б\u0069р\u0069нш\u0069", 1), ("б\u0456р\u0456нш\u0456", 1), ("еж", 1), ("ёж", 1)]` | merging Latin `i` with Kazakh `і`, or `ё` with `е`; splitting where the script changes |
| `"\u0435\u0308ж \u0451ж"` | `[("\u0451ж", 2)]` | deleting marks before NFC, giving `еж` |
| `"доро\u0301га дорога вс\u0435\u0300 все"` | `[("дорога", 2), ("все", 1), ("вс\u0450", 1)]` | keeping the acute mark or splitting at it; deleting a grave accent that NFC composes into `ѐ` |
| `"\u0130stanbul istanbul"` | `[("istanbul", 2)]` | keeping the U+0307 that `casefold()` adds |
| `"BILIM bılım Işkı ışkı"` | `[("bilim", 1), ("bılım", 1), ("işkı", 1), ("ışkı", 1)]` | Kazakh-Latin casing (`I` → `ı`), giving `[("bılım", 2), ("ışkı", 2)]` |
| `"№5 закон, 100 км² 100км 5½"` | `[("закон", 1), ("км", 1)]` | NFKC (`№` → `No`); `[^\W\d_]+`, which keeps `км²` and `½`; counting `км` in `100км` |
| `"пере\u00adход переход ca\u200dt cat\u200bdog"` | `[("cat", 2), ("переход", 2), ("dog", 1)]` | the soft hyphen or ZWJ as a separator; ZWSP joining words |
| `"don't don\u2018t don\u2019t don\u02bct don\u02bbt"` | `[("don't", 4), ("don\u02bbt", 1)]` | splitting at apostrophes; treating the letter `ʻ` as `'` |
| `"Trump's plan. Trump said"` | `[("plan", 1), ("said", 1), ("trump", 1), ("trump's", 1)]` | stripping the possessive `'s` |
| `"кое-что кое\u2011что well\u2010known well-known well known"` | `[("well-known", 2), ("кое-что", 2), ("known", 1), ("well", 1)]` | splitting at every hyphen, or joining only at ASCII `-` |
| `"'cat' -cat- a''b a--b"` | `[("a", 2), ("b", 2), ("cat", 2)]` | keeping edge `'` or `-`; joining across two joiners |
| `"a\u2013b a\u00b4b a\u0060b"` | `[("a", 3), ("b", 3)]` | treating U+2013, U+00B4 or U+0060 as joiners |
| `"covid19 covid_19 covid-19 F-16 20th 90's 1990-х 1-ші"` | `[("covid", 3), ("f", 1)]` | digits or `_` inside words; counting the suffixes `th`, `s`, `х` or `ші` |
| `"5%-ға «ҚТЖ»-ның (ҚТЖ)-ға"` | `[("қтж", 2)]` | counting `ға` or `ның` after `%`, a quote or `)` as words |
| `"АҚШ АҚШ-тың ҚР-да ҚР"` | `[("ақш", 1), ("ақш-тың", 1), ("қр", 1), ("қр-да", 1)]` | stripping endings that follow letters |
| `"-ке 5G-ге 100км-ге 152-ФЗ"` | `[("ке", 1)]` | dropping `ке` after a joiner at the start; keeping `ге` after `5G` or `100км`; keeping `фз` |
| `"Он ответил:\n-Да.\n-Не знаю.\u200b-ке"` | `[("да", 1), ("знаю", 1), ("ке", 1), ("не", 1), ("он", 1), ("ответил", 1)]` | dropping `да`, `не` or `ке` after a line break or U+200B and `-` |
| `"\u201c\u2018We\u2019re here\u2019 \u2014\u2018It\u2019s\u2019"` | `[("here", 1), ("it's", 1), ("we're", 1)]` | dropping a short word after an opening quote |
| `"20th-century 10-day 5млн 1-орын нан 10-нан said--no"` | `[("century", 1), ("day", 1), ("no", 1), ("said", 1), ("нан", 1)]` | dropping Latin words such as `century` or `day` after a number; counting `млн`, `орын` or the `нан` of `10-нан`; dropping `no` after `--` |
| `"мен «ҚТЖ»-мен a 2-to-1 vote"` | `[("a", 1), ("vote", 1), ("мен", 1), ("қтж", 1)]` | counting the `мен` of `«ҚТЖ»-мен`; keeping `to` after `2-` |
| `"men «QTJ»-men"` | `[("men", 2), ("qtj", 1)]` | treating a Latin run of three or more letters as an ending |
| `"U.S.-EU talks. Ответил:-Да."` | `[("s", 1), ("talks", 1), ("u", 1), ("ответил", 1)]` | keeping `eu` or `да` after `.` or `:` and `-` |
| `"\u2139\ufe0fcat 🙂cat"` | `[("cat", 1), ("\u2139cat", 1)]` | treating every emoji as a separator |
| `"U.K. т.е. https://inform.kz"` | `[("https", 1), ("inform", 1), ("k", 1), ("kz", 1), ("u", 1), ("е", 1), ("т", 1)]` | keeping abbreviations or URLs as one word |
| `"<b>cat</b> don&#8217;t"` | `[("b", 2), ("cat", 1), ("don", 1), ("t", 1)]` | stripping HTML tags or decoding entities |
| `"the cat the и және"` | `[("the", 2), ("cat", 1), ("және", 1), ("и", 1)]` | removing stop words |
| `"дорога дороги қазақ qazaq"` | `[("qazaq", 1), ("дорога", 1), ("дороги", 1), ("қазақ", 1)]` | stemming, or transliteration |
| `"ё я aa a"` | `[("a", 1), ("aa", 1), ("я", 1), ("ё", 1)]` | breaking ties by first-seen or alphabetical order (`ё` before `я`) |
| `"k j i h g f e d c b a"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` | returning all 11 tied words, or keeping `k` |
| `S("Cat")`, with `class S(str): casefold = __str__ = lambda self: "dog"` | `[("cat", 1)]` | rejecting subclasses, or calling their methods |
| `top_words("x").append(0)`, then `top_words("x")` | `[("x", 1)]` | returning a shared or cached list |
| `""`, `" \n\t2024 !? 🙂"` | `[]` for each | `None`, or a placeholder |
| `None`, `b"cat"` | raises `TypeError` | returning `[]`, or decoding bytes |
