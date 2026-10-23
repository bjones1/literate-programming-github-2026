# Specification: `top_words`

`top_words(text: str) -> list[tuple[str, int]]`, in `top_words.py` (Python 3.9+, standard library only), returns the 10 most common words of an article (English, Russian, Kazakh, or a mix) as `(word, count)` tuples, sorted by count descending, then by word ascending. With fewer than 10 distinct words, all are returned; with none, `[]`.

Notation: `cat: 3, dog: 2` means `[('cat', 3), ('dog', 2)]`; `\uXXXX` is a single code point.

## Algorithm (normative)

1. If `text` is not a `str` (subclasses are fine), raise `TypeError`.
2. Normalize to NFC (not NFKC).
3. Delete U+0301 (Russian stress mark) and U+00AD (soft hyphen).
4. Replace U+2019 and U+02BC with `'`.
5. Apply `str.lower()` (not `casefold()`).
6. Extract words. A *letter* is a character with `isalpha()`; a *mark* has Unicode category `M*`. A word starts with a letter and continues with letters and marks. A single `'` or `-` directly after a word's last character and directly before a letter joins the next piece onto the word (`a-b-c`, `rock'n'roll`). Everything else separates words: whitespace, punctuation, digits, `_`, other dashes, zero-width characters, a mark with no letter before it, and any `'` or `-` that is leading, trailing, or doubled.
7. Count, sort by `(-count, word)` using Python string comparison (code point order), and return the first 10. Returned words are the normalized, lowercased forms from steps 2-5.

## Decisions and examples

Each row shows an input whose output would differ under another plausible reading.

| Decision | Input | Expected output |
|---|---|---|
| Basic counting | `cat dog cat bird dog cat` | cat: 3, dog: 2, bird: 1 |
| No padding to 10 | `cat dog` | cat: 1, dog: 1 |
| No words gives `[]` | `123 !!!` | (empty) |
| Non-`str` input is an error | `None`, `b"cat"` | `TypeError` |
| Punctuation separates | `Hello, world! Hello.` | hello: 2, world: 1 |
| Digits and `_` separate | `abc123def snake_case` | abc: 1, case: 1, def: 1, snake: 1 |
| One-letter words count | `a a b` | a: 2, b: 1 |
| Other dashes and zero-width space separate | `cat\u2014dog cat\u200bdog` | cat: 2, dog: 2 |
| Marks stay in the word (no precomposed form) | `q\u0323a q\u0323a` | `q\u0323a`: 2 |
| A stray leading mark is dropped | `\u0323cat cat` | cat: 2 |
| Inner hyphen joins | `well-known well-known` | well-known: 2 |
| Inner hyphen joins (Russian) | `из-за дождя из-за` | из-за: 2, дождя: 1 |
| Inner apostrophe joins; ’ equals ' | `don't don\u2019t` | don't: 2 |
| Edge apostrophes are stripped | `'cat' cat dogs' dogs` | cat: 2, dogs: 2 |
| Edge and doubled hyphens separate | `-cat- cat--dog a-1` | cat: 2, a: 1, dog: 1 |
| Several joins allowed | `a-b-c a-b-c` | a-b-c: 2 |
| Two different joiners in a row separate | `cat-'dog` | cat: 1, dog: 1 |
| Only ASCII hyphen joins | `cat\u2010dog cat\u2011dog` | cat: 2, dog: 2 |
| Case-insensitive, all scripts | `Cat CAT cat Қазақстан ҚАЗАҚСТАН` | cat: 3, қазақстан: 2 |
| `lower()`, not `casefold()` | `Straße STRASSE` | strasse: 1, straße: 1 |
| `lower()` may create a mark, which stays | `\u0130stanbul istanbul` | istanbul: 1, `i\u0307stanbul`: 1 |
| NFC unifies equivalent spellings | `cafe\u0301 caf\u00e9` | café: 2 |
| Stress mark dropped | `молоко\u0301 молоко` | молоко: 2 |
| Soft hyphen dropped | `inter\u00adnational international` | international: 2 |
| Known exception: NFC composes г/к + U+0301 into ѓ/ќ before the mark is dropped | `\u0433\u0301 \u0433` | `\u0433`: 1, `\u0453`: 1 |
| No NFKC | `\uff43\uff41\uff54 cat` (full-width `ｃａｔ`) | cat: 1, `\uff43\uff41\uff54`: 1 |
| U+02BC maps to `'`; U+02BB does not | `o\u02bcz o'z o\u02bbz` | o'z: 2, `o\u02bbz`: 1 |
| No stop words | `the cat the dog the` | the: 3, cat: 1, dog: 1 |
| No stemming; `ё` is not `е` | `cat cats все всё все` | все: 2, cat: 1, cats: 1, всё: 1 |
| Look-alike letters are not merged | `a \u0430` (Latin a, Cyrillic а) | a: 1, `\u0430`: 1 |
| Possessives are separate words | `Putin's Putin` | putin: 1, putin's: 1 |
| Ties: ascending code point order | `pear apple pear apple fig` | apple: 2, pear: 2, fig: 1 |
| Latin before Cyrillic | `hello привет Hello Привет` | hello: 2, привет: 2 |
| Not locale order: `ё`, `ә` after `я` | `я ә а ё` | а: 1, я: 1, ё: 1, ә: 1 |
| `-` sorts before letters | `a-b ab` | a-b: 1, ab: 1 |
| Accented letters sort after ASCII | `élan zebra` | zebra: 1, élan: 1 |
| Limit of 10 cuts ties by the same order | `z z z a b c d e f g h i j` | z: 3, a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1 |
| No language detection | `Привет привет hello Hello сәлем` | hello: 2, привет: 2, сәлем: 1 |
| No segmentation of CJK | `\u4f60\u597d\u4e16\u754c` | `\u4f60\u597d\u4e16\u754c`: 1 |
| No markup or URL handling | `<b>cat</b> example.com` | b: 2, cat: 1, com: 1, example: 1 |

## Out of scope

Stop-word filtering, stemming, and word segmentation. Callers who filter the result afterwards get fewer than 10 useful words; supporting that would need an interface change (for example an `exclude` parameter) and a revised spec.
