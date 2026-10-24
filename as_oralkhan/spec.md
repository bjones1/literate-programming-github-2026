# Specification: `top_words`

## Interface

Implemented in `top_words.py`, Python 3:

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

- `text`: the article's plain text (not HTML — tags and entities are not
  stripped; if `text` came from HTML, strip it before calling this
  function), as a single `str`. It may mix English, Russian, and Kazakh in
  any proportion; Kazakh is assumed to be written in Cyrillic script. Any
  other type raises `TypeError` (`isinstance(text, str)` is the check, so a
  `str` subclass is accepted; `bytes`, `None`, and everything else is not).
- Returns: the 10 most frequent distinct casefolded words in `text` and
  their counts, as a list of `(word, count)` tuples, ranked most frequent
  first (decision 6 covers what happens with fewer than 10).

## Decisions

Each decision below resolves one place where the original one-paragraph
problem statement admits more than one reading. Every table gives an input,
this spec's output, and at least one different output a *different*
plausible reading of the original problem would produce instead.

### 1. What counts as a word

A word is a maximal run of Unicode letters (`c.isalpha()`), optionally
extended across an internal apostrophe or hyphen when a letter immediately
follows it. Before matching, the text is cleaned up in this order, so that
characters which are typographically or visually part of a word don't act
as accidental delimiters:

```python
import re, unicodedata

_WORD = re.compile(r"[^\W\d_]+(?:['\-][^\W\d_]+)*")

def _words(text: str) -> list[str]:
    text = unicodedata.normalize("NFC", text)
    text = text.translate({0x2019: "'", 0x2010: "-", 0x2011: "-", 0x200B: " "})
    text = "".join(c for c in text if unicodedata.category(c) not in ("Mn", "Cf"))
    text = "".join(" " if c.isalnum() and not c.isalpha() else c for c in text)
    return _WORD.findall(text)
```

1. **NFC-normalize**, so a letter typed as a base character plus a combining
   mark (e.g. `и` + combining breve) becomes the same single precomposed
   character (`й`) as if it had been typed directly.
2. **Fold typographic punctuation and the zero-width space to their plain
   form**: curly apostrophe `’` to `'`, the Unicode hyphen variants `‐`/`‑`
   to `-`, and U+200B ZERO WIDTH SPACE to an ordinary space — unlike the
   other invisible characters handled next, a zero-width space is sometimes
   a real (if invisible) word boundary, so it must become a separator, not
   simply vanish.
3. **Drop the remaining combining marks and invisible formatting
   characters** (Unicode categories Mn and Cf) — stress accents, soft
   hyphens, zero-width joiners — since these have no precomposed form for
   NFC to fold into, and are always attached *to* a word rather than
   separating two words, so dropping rather than spacing them is correct
   here.
4. **Turn non-letter alphanumerics into spaces**: `c.isalnum() and not
   c.isalpha()` is true only for digits and the handful of other Unicode
   "numeric" characters that aren't digits either — superscripts, vulgar
   fractions, Roman numerals. Doing this explicitly keeps the regex class
   `[^\W\d_]`, which is `\w` without digits/underscore, *exactly* equivalent
   to "is a letter": nothing numeric is left for it to accidentally match.

The regex itself then just finds runs of letters, allowing a single
`'`/`-` between two letter-runs to glue them into one word. A leading,
trailing, or doubled hyphen/apostrophe is a delimiter, not part of a word.
There is no minimum word length: a single letter standing alone (English
`a`/`I`, Russian `и`/`в`) is a valid one-letter word.

| Input                               | Expected output                          | A different plausible reading |
| -------------------------------------- | --------------------------------------------- | --------------------------------------------------------- |
| `"Hello world! Привет мир. Сәлем, әлем."` | `hello: 1, world: 1, мир: 1, привет: 1, сәлем: 1, әлем: 1` | Matching only `[a-zA-Zа-яА-Я]` would mangle `сәлем` into `с` + `лем`. |
| `"covid19 won 2026"`                   | `covid: 1, won: 1`                            | Treating digits as word characters would instead produce `covid19` and `2026` as words. |
| `"don't stop state-of-the-art"`        | `don't: 1, state-of-the-art: 1, stop: 1`      | Splitting on every apostrophe/hyphen would instead produce `don`, `t`, `state`, `of`, `the`, `art`. |
| `"well--known"` (a doubled apostrophe, e.g. `don''t`, is delimited the same way) | `known: 1, well: 1` | Treating a doubled hyphen the same as a single one would instead produce `well-known` as one word. |
| `"площадь 100 км² и 5 м³"`             | `и: 1, км: 1, м: 1, площадь: 1`               | Matching Unicode-numeric characters as word characters (step 4 skipped) would instead produce `км²` and `м³` as single words. |
| `"don't don’t"` (straight `'`, then curly `’`) | `don't: 2`                            | Not folding `’` to `'` (step 2 skipped) would instead produce two different one-count words, `don't` and `don’t`. |
| `"по‑русски по-русски"` (first uses U+2011, non-breaking hyphen) | `по-русски: 2`           | Not folding hyphen variants would instead produce `по-русски: 1` twice over as different words. |
| `"за́мок замок"` (first has a combining stress accent) | `замок: 2`               | Leaving Mn characters in place (step 3 skipped) would instead produce `за: 1, мок: 1, замок: 1`. |
| `"чай ча" + "и\u0306"` (a Python expression: renders identically to plain `"чай чай"`, but the second word's final letter is `и` + U+0306 COMBINING BREVE, not precomposed `й` — expressed this way because decomposed Unicode does not survive being pasted as plain text through most editors) | `чай: 2`  | Skipping NFC (step 1) would instead produce `чай: 1, чаи: 1` — the combining breve becomes an invisible delimiter, so the decomposed word silently loses its last letter. |

### 2. Case folding

Every word is folded with `str.casefold()`, not `.lower()`, since
`casefold()` also normalizes cases `lower()` misses (e.g. German `ß` →
`ss`). One consequence: a returned word is not always a literal substring of
`text` — `Straße` becomes `strasse`, and ligatures like `ﬁ` fold to `fi`.
This is intentional, so that case-only and compatibility-only variants of
the same word count as one word.

| Input                | Expected output   | A different plausible reading                          |
| --------------------- | -------------------- | --------------------------------------------------------- |
| `"Cat cat CAT"`       | `cat: 3`              | Case-sensitive counting would instead produce three separate one-count words: `Cat`, `cat`, `CAT`. |
| `"Ізде із ІЗДЕ"`      | `ізде: 2, із: 1`      | Same as above, applied to Kazakh Cyrillic casing.          |
| `"Straße STRASSE"`    | `strasse: 2`          | Using `.lower()` instead would produce `straße: 1, strasse: 1` — two different words. |

### 3. Stop words are not removed

No language's stop words (`the`, `и`, `және`, ...) are filtered out — doing
so would require inventing a per-language stop-word list, itself a second,
unstated spec, and the original prompt never mentions removing them. One
consequence worth stating plainly: for a real article, the top 10 will
usually be dominated by function words (`the, of, and` / `и, в, на` /
`және, мен`), and since single letters count as words (decision 1),
abbreviations contribute their letters too.

| Input                                   | Expected output                          | A different plausible reading                                 |
| ----------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------ |
| `"the cat and the dog and the bird"`     | `the: 3, and: 2, bird: 1, cat: 1, dog: 1`     | Removing English stop words would drop `the` and `and` entirely. |
| `"и т.д. и т.е."`                        | `и: 2, т: 2, д: 1, е: 1`                      | Removing single-letter "words" would instead produce `[]`. |

### 4. Inflectional forms are not normalized

Russian "дом", "дома", "дому", "домом" are four surface forms of one root;
Kazakh adds its own case suffixes the same way. This spec counts exact
surface forms, not lemmas — stemming/lemmatizing is, like decision 3, a
second per-language spec of its own that typically needs a dictionary or
model beyond the standard library.

| Input                          | Expected output                       | A different plausible reading |
| --------------------------------- | ------------------------------------------ | ----------------------------------------------------------- |
| `"дом дома дому домом"`          | `дом: 1, дома: 1, домом: 1, дому: 1`      | Lemmatizing to a shared root would instead produce `дом: 4`. |

### 5. Ranking and tie-breaking

Results are ordered by count descending; ties are broken by the casefolded
word, ascending in **Unicode code-point order** (not locale-aware
alphabetical order). Code-point order puts `ё` (U+0451) after the plain
Russian alphabet `а`–`я` (U+0430–U+044F), but *before* all nine
Kazakh-specific letters (`ә ғ қ ң ө ұ ү һ і`, U+0493 and above) — a
different order from either Russian collation (which puts `ё` right after
`е`) or Kazakh collation. This also decides which words make the top 10
when a count tie would otherwise straddle the cutoff: the full list is
sorted first, by count then by code point, and the top 10 of that single
sorted list are kept — so if a tie only partly fits in the remaining slots,
only the code-point-earliest of the tied words fill those slots.

| Input                                                                 | Expected output                                                               | A different plausible reading |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `"zebra apple mango banana cherry date eggplant fig grape honeydew kiwi"` (each word appears once) | `apple: 1, banana: 1, cherry: 1, date: 1, eggplant: 1, fig: 1, grape: 1, honeydew: 1, kiwi: 1, mango: 1` (code-point order; `zebra` dropped) | First-appearance order would instead keep the first 10 words seen — `zebra, apple, mango, banana, cherry, date, eggplant, fig, grape, honeydew` — dropping `kiwi` instead of `zebra`. |
| `"ёж яблоко"`                                                           | `яблоко: 1, ёж: 1`                                                               | Locale-collated Russian alphabetical order would instead give `ёж: 1, яблоко: 1`. |

### 6. Fewer than 10 words, and no words at all

If `text` contains between 1 and 9 distinct casefolded words, all of them
are returned (no padding with `None` or zero-count placeholders). If `text`
contains zero words — the empty string, whitespace-only text, or text made
up entirely of digits/punctuation — `top_words` returns `[]` rather than
raising an exception.

| Input           | Expected output      | A different plausible reading                          |
| ------------------ | ---------------------- | ----------------------------------------------------------- |
| `"one one two"`    | `one: 2, two: 1`       | Padding to a fixed length of 10 would instead return 8 extra `None`/`("", 0)` entries. |
| `""`                | `[]`                    | Raising `ValueError` on empty input is an equally plausible but different reading of "no input". |
| `"123 !!! ---"`     | `[]`                    | Same as above — no letters means no words, not an error.    |

### 7. Input validation

If `text` is not a `str` (checked with `isinstance`, so `str` subclasses are
accepted), `top_words` raises `TypeError`. There is no silent coercion.

| Input   | Expected output   | A different plausible reading |
| --------- | -------------------- | ----------------------------------------------------------- |
| `None`    | raises `TypeError`   | Treating `None` as empty text would instead give `[]`. |
| `b"cat"`  | raises `TypeError`   | Decoding `bytes` as UTF-8 would instead give `cat: 1`. |
