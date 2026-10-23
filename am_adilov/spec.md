# Specification: Top words

A news site shows a "Trending words" box beside each article. `top_words`
returns the ten most common words in an article's text, with their counts.
Articles may be in English, Russian, or Kazakh.

## Interface

```python
def top_words(text: str) -> list[tuple[str, int]]: ...
```

- Standard library only. The function is named `top_words` and lives in
  `top_words.py`.
- Returns a list of `(word, count)` pairs, highest count first, ties broken as
  in rule 16, holding at most ten pairs.
- Raise `TypeError` when `isinstance(text, str)` is false. A `str` subclass is
  accepted, and the words in the returned pairs are plain `str`.
- No other exception is raised: every `str` has an answer, including `""` and
  strings holding unpaired surrogates.
- Conformance is judged on CPython 3.13 (`unicodedata.unidata_version`
  `15.1.0`), which the examples were produced on. Results may differ on
  another version only for characters whose category, case folding or
  decomposition changed; no example here uses such a character. Nothing else
  is version-gated: the annotations above need Python 3.9 or later, and the
  rules use no newer syntax or library behavior.

## The pipeline

1. In order: normalize to NFC, delete combining marks inside words, split into
   words, casefold each word, re-normalize to NFC, delete combining marks
   again, fold `ё`, trim joiners, count, sort, cut to ten. The rules below
   fill in each step; where two steps could disagree, this order decides.

## Preparing the text

2. `text` is plain article text. Stripping HTML is the caller's job, and this
   function does not attempt it, so markup passed in makes tag names into
   trending words.
3. The text is normalized to Unicode NFC first. This matters for the two
   letters Russian and Kazakh share that have a decomposed spelling: `й`
   (U+0439) typed as `и` + U+0306, and `ё` (U+0451) typed as `е` + U+0308 are
   composed to their single-code-point forms. The nine Kazakh-specific letters
   (`ә ғ қ ң ө ұ ү һ і`) have no decomposition, so NFC leaves them alone.
4. Combining marks inside words are then deleted, left to right: a character
   of category `Mn` or `Mc` is deleted when the nearest preceding character
   that has not itself been deleted is alphabetic. A run of marks after a
   letter is therefore deleted entirely, however long. These are the marks NFC
   could not compose, and deleting them keeps a marked word whole instead of
   splitting it: Russian stress marking (`мо́лодец`, U+0301) counts as the
   unmarked word, and a hand-rolled `ө` written as `о` + U+0335 counts as `о`.
   A mark with no such preceding character is left alone and separates words
   under rule 11.

## What counts as a word

5. A character is **alphabetic** when `ch.isalpha()` is true — exactly Unicode
   general category `L*` — and it is not one of the joiners in rule 6. The
   derived Unicode Alphabetic property is slightly wider, but not for any
   character in modern Latin, Cyrillic or Kazakh orthography, and the stdlib
   `re` module cannot match it, so `isalpha()` is the operational test.
6. The **joiners** are exactly six characters: `-` (U+002D), `'` (U+0027),
   `’` (U+2019), and the modifier letters `ʻ` (U+02BB), `ʼ` (U+02BC) and
   `ʽ` (U+02BD) used for the apostrophe in Turkic Latin orthographies. The
   three modifier letters are category `Lm` and so would be alphabetic by
   rule 5; rule 5 excludes them so that they behave like the apostrophes they
   are.
7. Words are found by scanning left to right. At each position, if a match of
   `A+(J A+)*` starts there — `A` an alphabetic character, `J` a single joiner
   — the **longest** such match is a word, and scanning resumes at the first
   character after it; otherwise scanning advances one character. Words
   therefore never overlap: `"cat"` is the single word `cat`, not `cat`, `at`
   and `t`.
8. A joiner is part of a word only between two alphabetic characters, any
   number of times, so `кто-то-нибудь` is one word.
9. It follows that a leading or trailing joiner is not part of the word, and
   that two or more joiners in a row — in any mixture, `--` or `-'` alike —
   split the word. (Casefolding can reintroduce a leading joiner; rule 14
   trims it.)
10. Digits are not alphabetic, so they are separators: `2026` yields nothing,
    `a1b` yields `a` and `b`, and `covid-19` yields just `covid` — its hyphen
    has a digit on one side, so it is trailing rather than internal.
11. Every other character is a separator: space, newline, punctuation, emoji,
    and any combining mark left by rule 4.
12. The hyphen set is deliberately ASCII-only: U+2010 HYPHEN, U+2011
    NON-BREAKING HYPHEN and U+00AD SOFT HYPHEN are separators, not joiners.
    A soft hyphen from justified typesetting is invisible on screen and splits
    the word around it; strip it upstream if that matters.

## How words are grouped

13. Words are compared after `str.casefold()`. Casefolding is not a plain
    lowercasing: it can change a word's length (`ß` → `ss`), emit a combining
    mark, which cannot occur in a word (`İ` → `i` + U+0307), and emit a
    joiner in a position a word cannot start with (`ŉ` → `ʼ` + `n`).
    Rules 1 and 4 then re-normalize and delete the combining marks, so
    `İnternet` groups with `internet` rather than standing alone — which
    matters because Kazakh Latin writes `İ` at the start of a sentence and
    `i` inside one. Dotless `I` is a known limitation: it casefolds to `i`,
    not to `ı`, so a sentence-initial dotless-`I` word does not group with
    its `ı` forms. Folding it the other way would break English `I`, and
    telling the two apart needs language detection, which is out of scope.
14. Every U+0451 (`ё`), in any language, is replaced by U+0435 (`е`). Leading
    and trailing joiners are then trimmed, which is what turns casefolded
    `ŉ` into the word `n`. The spelling returned is this fully folded one:
    `ещё` is reported as `еще` even when `ещё` is the only spelling in the
    article. A word whose folded form is empty is dropped.
15. The `ё` fold is the only *language-specific* folding. Kazakh `і`/`и`,
    `ү`/`у` and similar look-alike pairs stay distinct words.

## Which ten, and in what order

16. Sort by count descending; break ties by the word itself, ascending by
    Unicode code point. Comparison is lexicographic over the full folded
    spelling, joiners included, so a word sorts before any word extending it
    (`кот` before `коты`) and `кот-а` sorts before `коты`, U+002D being below
    `ы`. Comparison is never locale-sensitive.
17. Keep the first ten pairs and discard the rest. Ties are cut by rule 16's
    order, not by input order, so when more words tie than there is room for,
    which of them survive is still fully determined: in the eleven-way tie of
    the `"a b c d e f g h i j k"` row, `k` is the word dropped.
18. Return every word when the article has ten or fewer distinct words; the
    list is not padded. In particular, text with no words at all gives `[]`.
19. No stop-word removal and no minimum length. `the`, `и`, `және` and the
    single-letter `с` are words like any other, so the box is pure frequency.
    Dropping them needs a per-language word list and is a product decision,
    out of scope here.
20. The result is therefore fully determined by the input — independent of
    locale, platform, and dictionary or input order.

## Examples

Each row pins at least one decision above, and most would come out differently
under another plausible reading of the original one-sentence problem. Inputs
and outputs are written as Python literals, so `\n` is a newline and `\u00ad`
is a soft hyphen. Every output was produced by a reference implementation of
these rules on CPython 3.13.

| Input | Expected output | Decisions pinned |
| --- | --- | --- |
| `"cat"` | `[("cat", 1)]` | words do not overlap (7) |
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | counting; fewer than ten words (18) |
| `"The the THE"` | `[("the", 3)]` | casefolding, and the folded spelling as output (13) |
| `"Кот кот КОТ собака"` | `[("кот", 3), ("собака", 1)]` | casefolding applies to Cyrillic (13) |
| `""` | `[]` | empty text (18) |
| `"2026 -- !!! 42"` | `[]` | digits and punctuation are not words (10, 11, 18) |
| `"a1b"` | `[("a", 1), ("b", 1)]` | a digit between letters splits (10) |
| `"covid-19 covid"` | `[("covid", 2)]` | `covid-19` is not a word; its hyphen is trailing (10) |
| `"covid-19-vaccine"` | `[("covid", 1), ("vaccine", 1)]` | the digit breaks the joiner chain both ways (8, 10) |
| `"что-то что-то что"` | `[("что-то", 2), ("что", 1)]` | an internal hyphen joins; the word is not split in two (8) |
| `"кто-то-нибудь"` | `[("кто-то-нибудь", 1)]` | more than one joiner per word is allowed (8) |
| `"-cat- cat"` | `[("cat", 2)]` | leading and trailing joiners are dropped (9) |
| `"re--do re do"` | `[("do", 2), ("re", 2)]` | a doubled joiner splits; tie broken by code point (9, 16) |
| `"re-'do re do"` | `[("do", 2), ("re", 2)]` | a *mixed* joiner pair splits too (9) |
| `"don't don’t"` | `[("don't", 1), ("don’t", 1)]` | both apostrophes join, but they are different characters, so these are different words (6, 16) |
| `"qaʼla ʼqala"` | `[("qala", 1), ("qaʼla", 1)]` | U+02BC is a joiner, not a letter: internal it stays, leading it is dropped (6, 9) |
| `"qaʻla ʻqala"` | `[("qala", 1), ("qaʻla", 1)]` | U+02BB behaves identically — the Uzbek/Karakalpak form editors substitute (6) |
| `"ŉ ʼn"` | `[("n", 2)]` | casefolding `ŉ` emits a leading joiner, which rule 14 trims (13, 14) |
| `"ещё еще"` | `[("еще", 2)]` | `ё` folds to `е` (14) |
| `"ЁЖ ёж"` | `[("еж", 2)]` | uppercase `Ё` folds too, because casefolding runs first (1, 14) |
| `"мо\u0301лодец молодец"` | `[("молодец", 2)]` | a Russian stress mark is deleted, not treated as a separator (4) |
| `"о\u0335й"` | `[("ой", 1)]` | likewise for a mark NFC cannot compose (4) |
| `"мо\u0335\u0301лодец молодец"` | `[("молодец", 2)]` | a whole run of marks is deleted, not just the first (4) |
| `"и\u0306 й"` | `[("й", 2)]` | NFC composes the decomposed `й` (3) |
| `"İnternet internet"` | `[("internet", 2)]` | `İ` casefolds to `i` + U+0307, whose mark is then deleted (13) |
| `"үй уй"` | `[("уй", 1), ("үй", 1)]` | Kazakh `ү` and `у` stay distinct; `у` (U+0443) sorts before `ү` (U+04AF) (15, 16) |
| `"cat\u2011dog"` | `[("cat", 1), ("dog", 1)]` | a non-breaking hyphen is a separator, not a joiner (12) |
| `"soft\u00adhyphen"` | `[("hyphen", 1), ("soft", 1)]` | an invisible soft hyphen splits the word (12) |
| `"<b>cat</b>"` | `[("b", 2), ("cat", 1)]` | no markup stripping: the top trending word is `b` (2) |
| `"cat\U0001f642dog\ncat"` | `[("cat", 2), ("dog", 1)]` | emoji and newlines separate (11) |
| `"the cat and the dog"` | `[("the", 2), ("and", 1), ("cat", 1), ("dog", 1)]` | no stop-word removal (19) |
| `"мы с тобой"` | `[("мы", 1), ("с", 1), ("тобой", 1)]` | no minimum length: `с` counts (19) |
| `"b c a"` | `[("a", 1), ("b", 1), ("c", 1)]` | an all-tie result is alphabetical, not input order (16) |
| `"кот коты кот-а"` | `[("кот", 1), ("кот-а", 1), ("коты", 1)]` | a prefix sorts first; comparison is by raw code point, joiners included, and U+002D is below `ы` (16) |
| `"a b c d e f g h i j k"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` | eleven tied words cut to ten by code point; `k` is dropped (16, 17) |
| `"\ud800"` | `[]` | an unpaired surrogate is a separator, not an error (interface, 11) |
| `MyStr("a b a")`, a `str` subclass | `[("a", 2), ("b", 1)]` | subclasses are accepted (interface) |
| `42` (an `int`) | raises `TypeError` | non-`str` input (interface) |

## Worked tie-break

`"b a c a b"` gives `a: 2`, `b: 2`, `c: 1`. Sorting by count descending puts
`a` and `b` ahead of `c`; the tie between them goes to `a`, which sorts first.
Result: `[("a", 2), ("b", 2), ("c", 1)]` — not `[("b", 2), ("a", 2), ("c", 1)]`,
which is what first-seen order would give.
