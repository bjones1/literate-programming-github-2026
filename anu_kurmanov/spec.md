# Top words: specification

Author: anu_kurmanov

A news site shows a "Trending words" box beside each article. This document
specifies a Python function that finds the 10 most common words in an article's
text, along with how many times each one appears. Articles may be in English,
Russian, or Kazakh (Cyrillic or Latin script).

Every decision below is followed by examples. In the tables, the input column is
the argument, written as a Python literal (almost always a string). The expected
output column is either the exact value the function returns, written as a
Python literal, or "raises" and the exception type. Each example was chosen so that
some other plausible reading of the original problem gives a different result.

## 1. Interface

- Language: Python 3.10 or newer, standard library only. No third-party packages
  (no `regex`, `nltk`, `pymorphy`, etc.).
- Which characters are letters and marks, and how NFC and `str.lower()` behave,
  is defined by the Unicode database of the running interpreter
  (`unicodedata.unidata_version`). The implementation must not ship its own
  character tables. Every example in this document uses only characters whose
  properties are the same in Unicode 13.0 (Python 3.10) and later.
- File: `top_words.py`. Function: `top_words`.
- Signature: `def top_words(text: str) -> list[tuple[str, int]]:`
- The function is pure: it does not read files, print, use the network, or keep
  state between calls. The same input always gives the same output on the same
  interpreter (the same Unicode database, see the next bullet).
- The result is a `list` of at most 10 `tuple`s. Each tuple is `(word, count)`,
  where `word` is a `str` and `count` is an `int` of at least 1.
- If `text` is not a `str` (for example `None`, `bytes`, or a list), the
  function raises `TypeError`. It never returns `None`. A subclass of `str` is
  accepted: it is first converted to a plain `str` with `str.__str__(text)`, so
  any methods the subclass overrides (`lower`, `__iter__`, `__str__`, ...) are
  never used. The returned words are always plain `str` objects. For example,
  given `class Shout(str): lower = str.upper`, `top_words(Shout("Hi hi"))`
  returns `[("hi", 2)]`, and `type(top_words(Shout("Hi"))[0][0]) is str`.
  Overriding `__str__` has no effect either: given
  `class W(str): __str__ = lambda self: "zzz"`, `top_words(W("Hi hi"))` returns
  `[("hi", 2)]`, while an implementation that called `str(text)` would wrongly
  return `[("zzz", 1)]`.

| Input                        | Expected output                           |
| ---------------------------- | ----------------------------------------- |
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]`   |
| `""`                         | `[]`                                      |
| `"   \n\t "`                 | `[]`                                      |
| `"!!! ... 2026 --- ?"`       | `[]`                                      |
| `None`                       | raises `TypeError`                        |
| `b"cat"`                     | raises `TypeError`                        |
| `["cat", "dog"]`             | raises `TypeError`                        |

The result is a list of tuples, not a `dict`, a `Counter`, or a string such as
`"cat: 3, dog: 2, bird: 1"`. Text with no words gives an empty list, not an
error.

## 2. Unicode normalization

After the type check and conversion to plain `str` (section 1), the text is
converted to Unicode normalization form NFC
(`unicodedata.normalize("NFC", text)`). So a letter typed as one precomposed
character and the same letter typed as a base letter plus a combining mark are
the same word.

| Input                                  | Expected output |
| -------------------------------------- | --------------- |
| `"\u0451\u0436 \u0435\u0308\u0436"`    | `[("ёж", 2)]`   |
| `"\u0439 \u0438\u0306"`                | `[("й", 2)]`    |

These rows are written with `\u` escapes because the two spellings look the
same on screen. In the first row, `ёж` is written once with precomposed `ё`
(U+0451) and once with `е` + combining diaeresis (U+0435 U+0308). In the second
row, `й` is written once precomposed (U+0439) and once as `и` + combining breve
(U+0438 U+0306). Without normalization each row would give two different
words.

Apart from lowercasing (section 3), no other folding happens: `ё` and `е` are different letters, and so are Kazakh
letters and the Russian letters they look like (`қ` and `к`, `ү` and `у`, `і`
and `и`, `ә` and `а`, `ө` and `о`, `ғ` and `г`, `ұ` and `у`, `ң` and `н`, `һ`
and `х`). Accents are not removed (`café` stays `café`). Nothing is
transliterated between Cyrillic and Latin.

| Input                    | Expected output                       |
| ------------------------ | ------------------------------------- |
| `"ёж еж ёж"`             | `[("ёж", 2), ("еж", 1)]`              |
| `"қазақ казак қазақ"`    | `[("қазақ", 2), ("казак", 1)]`        |
| `"café cafe café"`       | `[("café", 2), ("cafe", 1)]`          |
| `"Қазақстан Qazaqstan"`  | `[("qazaqstan", 1), ("қазақстан", 1)]` |

## 3. Case

Words are compared and returned in lowercase, using Python's `str.lower()`
applied to each word separately (not to the whole text), and then NFC is applied
to the lowered word again. (`str.lower()`, not `str.casefold()`: `"Straße"` becomes
`"straße"`, not `"strasse"`.)

| Input                         | Expected output                  |
| ----------------------------- | -------------------------------- |
| `"The the THE tHe"`           | `[("the", 4)]`                   |
| `"Алматы АЛМАТЫ алматы"`      | `[("алматы", 3)]`                |
| `"Әке әке ӘКЕ"`               | `[("әке", 3)]`                   |
| `"Straße STRASSE"`            | `[("strasse", 1), ("straße", 1)]` |
| `"J\u030c \u01f0"`            | `[("\u01f0", 2)]`                 |
| `"\u0391\u03a3.\u0392"`        | `[("\u03b1\u03c2", 1), ("\u03b2", 1)]` |

Row 5 shows why NFC is applied again: `J` + combining caron (U+030C) has no
precomposed capital, but lowering gives `j` + U+030C, which NFC composes into
`ǰ` (U+01F0). Without the second NFC, the result would be two entries that look
the same. Row 6 (`ΑΣ.Β`) shows why lowering is per word: the word `ΑΣ` lowers
to `ας` with final sigma `ς` (U+03C2). Lowering the whole text `ΑΣ.Β` would
give `σ` (U+03C3): `.` is case-ignorable and a cased letter follows it, so
Python does not treat `Σ` as word-final.

Lowering applies to the whole word, joiners included. Because `'` and `’` are
case-ignorable and `-` is not, the joiner changes the sigma:

| Input                                      | Expected output                                    |
| ------------------------------------------ | -------------------------------------------------- |
| `"\u0391\u03a3'\u0392 \u0391\u03a3-\u0392 \u0391\u03a3\u2019\u0392"` | `[("\u03b1\u03c3'\u03b2", 2), ("\u03b1\u03c2-\u03b2", 1)]` |

That is, `ΑΣ'Β` and `ΑΣ’Β` both become `ασ'β`, and `ΑΣ-Β` becomes `ας-β`.
Lowering each piece between joiners separately would give `ας'β` instead.

Proper nouns are not treated specially: `"Apple"` and `"apple"` are the same
word.

There is no locale-specific (Turkish/Azerbaijani-style) case mapping, even for
Kazakh in Latin script. `str.lower()` turns `I` into `i` and `İ` (U+0130) into
two code points, `i` + combining dot above (U+0307). Dotless `ı` (U+0131) stays
`ı`. So the rows below give two words each; an implementation must not merge
them.

| Input                    | Expected output                                    |
| ------------------------ | -------------------------------------------------- |
| `"\u0130L\u0130M ilim"`  | `[("ilim", 1), ("i\u0307li\u0307m", 1)]`            |
| `"IS \u0131s"`           | `[("is", 1), ("\u0131s", 1)]`                      |

## 4. What is a word

A **letter** is any character `c` for which `c.isalpha()` is `True`. This covers
Latin, Cyrillic (including all Kazakh letters such as `ә ғ қ ң ө ұ ү һ і`), and
every other script. A **mark** is a character whose Unicode category
(`unicodedata.category(c)`) is `Mn` or `Mc` (a combining mark). A **joiner** is
one of these four characters:

- apostrophe `'` (U+0027) or right single quotation mark `’` (U+2019);
- hyphen-minus `-` (U+002D) or hyphen `‐` (U+2010).

A **word** is a maximal run of characters that:

1. starts with a letter;
2. continues with letters and marks;
3. may contain any number of joiners, but never two in a row, and only where
   the character immediately before the joiner is a letter or mark that is
   already part of the word and the character immediately after it is a
   letter.

A mark is part of a word only when it comes after a letter or after another mark
that is part of the word. A mark anywhere else (at the start of the text, after
a space, or after a joiner) is a separator, like any other non-word
character. It is not deleted before scanning.

Every other character ends the current word and is not part of any word. That
includes spaces, line breaks, digits, underscores, punctuation, en dash `–`,
em dash `—`, quotation marks other than the two apostrophes above, symbols, and
emoji. A joiner is part of a word only when the character right before it is
part of that same word (a letter or a mark already in the word) and the
character right after it is a letter. Any other joiner is just a separator.
Only `Mn` and `Mc` marks count; enclosing marks (category `Me`, such as U+20DD
COMBINING ENCLOSING CIRCLE) are separators.

After a word is found, every `’` in it is replaced by `'` and every `‐` by `-`,
so the two spellings count as the same word.

Only these four characters are joiners. Look-alikes follow the general rules:
the soft hyphen (U+00AD) and the non-breaking hyphen (U+2011) are separators,
so they split a word. The modifier letter apostrophe `ʼ` (U+02BC) is a letter
(`isalpha()` is `True`), so it stays in the word unchanged and is not replaced
by `'`.

| Input                          | Expected output                                 |
| ------------------------------ | ----------------------------------------------- |
| `"inter\u00adnational"`        | `[("inter", 1), ("national", 1)]`               |
| `"non\u2011stop"`              | `[("non", 1), ("stop", 1)]`                     |
| `"doesn\u02bct doesn't"`       | `[("doesn't", 1), ("doesn\u02bct", 1)]`         |

### 4.1 Punctuation and separators

| Input                                 | Expected output                                 |
| ------------------------------------- | ----------------------------------------------- |
| `"Hello, world! Hello... (world)"`    | `[("hello", 2), ("world", 2)]`                  |
| `"«Привет», — сказал он. «Привет»!"`  | `[("привет", 2), ("он", 1), ("сказал", 1)]`     |
| `"snake_case_name"`                   | `[("case", 1), ("name", 1), ("snake", 1)]`      |
| `"a/b a/b"`                           | `[("a", 2), ("b", 2)]`                          |
| `"word🙂word"`                        | `[("word", 2)]`                                 |

### 4.2 Digits

Digits are never part of a word, and a number on its own is not a word. A digit
inside a run of letters splits it.

| Input                       | Expected output                       |
| --------------------------- | ------------------------------------- |
| `"2026 2026 year"`          | `[("year", 1)]`                       |
| `"COVID-19 covid"`          | `[("covid", 2)]`                      |
| `"mp3 mp4"`                 | `[("mp", 2)]`                         |
| `"5G and 4G"`               | `[("g", 2), ("and", 1)]`              |

In `"COVID-19"`, the hyphen has a digit after it, so it is a separator and the
word is `covid`.

### 4.3 Hyphens

A hyphen with a letter or mark of the same word before it and a letter after it
joins them into one word, and a word may contain several hyphens. Two hyphens in a row, a
hyphen next to a space, and dashes (`–`, `—`) all separate.

| Input                                | Expected output                                   |
| ------------------------------------ | ------------------------------------------------- |
| `"кое-как кое как"`                  | `[("как", 1), ("кое", 1), ("кое-как", 1)]`        |
| `"well-known well‐known"`            | `[("well-known", 2)]`                             |
| `"e-mail email"`                     | `[("e-mail", 1), ("email", 1)]`                   |
| `"Алматы-Астана"`                    | `[("алматы-астана", 1)]`                          |
| `"mother-in-law rock'n'roll"`        | `[("mother-in-law", 1), ("rock'n'roll", 1)]`      |
| `"up--down up - down"`               | `[("down", 2), ("up", 2)]`                        |
| `"war–peace war—peace"`              | `[("peace", 2), ("war", 2)]`                      |
| `"-start end-"`                      | `[("end", 1), ("start", 1)]`                      |
| `"inter-\nnational"`                 | `[("inter", 1), ("national", 1)]`                 |

The second row uses U+2010 in the second word; it counts the same as `-`. Line
break hyphenation is not undone: the last row is two words.

### 4.4 Apostrophes

An apostrophe with a letter or mark of the same word before it and a letter
after it is part of the word, and a word may contain several apostrophes.
Leading and trailing apostrophes, and apostrophes used as quotation marks, are separators.

| Input                                 | Expected output                                   |
| ------------------------------------- | ------------------------------------------------- |
| `"don't dont don't"`                  | `[("don't", 2), ("dont", 1)]`                     |
| `"don’t don't"`                       | `[("don't", 2)]`                                  |
| `"'quoted' quoted"`                   | `[("quoted", 2)]`                                 |
| `"the dogs' bowl"`                    | `[("bowl", 1), ("dogs", 1), ("the", 1)]`          |
| `"it's it is"`                        | `[("is", 1), ("it", 1), ("it's", 1)]`             |
| `"O'Brien o'brien"`                   | `[("o'brien", 2)]`                                |
| `"don\u2018t"`                        | `[("don", 1), ("t", 1)]`                          |
| `"don\u0060t"`                        | `[("don", 1), ("t", 1)]`                          |

The last two rows use the left single quotation mark `‘` (U+2018) and the
backtick (U+0060). They look like apostrophes but are not joiners, so they
split the word.

The 2017 Kazakh Latin alphabet wrote some letters with an apostrophe (for
example `s'` for `ш`). These spellings get no special handling: a word-final
apostrophe is a separator like any other, so `as'` counts as `as`. Kazakh Latin
in the 2021 alphabet uses letters with diacritics (`ä ğ ı ñ ö ş ū ü`), which are
ordinary letters.

| Input                                 | Expected output                                   |
| ------------------------------------- | ------------------------------------------------- |
| `"as' as"`                            | `[("as", 2)]`                                     |
| `"s'ah sah"`                          | `[("s'ah", 1), ("sah", 1)]`                       |
| `"Şaş şaş"`                           | `[("şaş", 2)]`                                    |

Contractions are not expanded: `it's` is a word of its own and does not add to
`it` or `is`.

### 4.5 Marks and single letters

A combining mark after a letter (or after another mark in the word) stays in the
word, whether it is a non-spacing (`Mn`) or spacing (`Mc`) mark. A combining
mark anywhere else is a separator (section 4). A joiner after a mark works the
same as after a letter. Single-letter words count like any other word.

| Input                          | Expected output                              |
| ------------------------------ | -------------------------------------------- |
| `"\u043c\u043e\u043b\u043e\u043a\u043e\u0301 \u043c\u043e\u043b\u043e\u043a\u043e"` | `[("\u043c\u043e\u043b\u043e\u043a\u043e", 1), ("\u043c\u043e\u043b\u043e\u043a\u043e\u0301", 1)]` |
| `"\u043c\u043e\u043b\u043e\u043a\u043e\u0301-\u0442\u043e"` | `[("\u043c\u043e\u043b\u043e\u043a\u043e\u0301-\u0442\u043e", 1)]` |
| `"\u0301abc"`                  | `[("abc", 1)]`                               |
| `"a-\u0301b"`                  | `[("a", 1), ("b", 1)]`                       |
| `"a-\u0301"`                   | `[("a", 1)]`                                 |
| `"1\u0301-b"`                  | `[("b", 1)]`                                 |
| `"a\u20ddb"`                   | `[("a", 1), ("b", 1)]`                       |
| `"a\u0301\u0302 a"`            | `[("a", 1), ("\u00e1\u0302", 1)]`            |
| `"\u0915\u093e \u0915"`        | `[("\u0915", 1), ("\u0915\u093e", 1)]`        |
| `"я и я и я"`                  | `[("я", 3), ("и", 2)]`                       |
| `"a I a"`                      | `[("a", 2), ("i", 1)]`                       |

The first two rows are `молоко́ молоко` and `молоко́-то`, written with escapes.
The stress mark (U+0301) has no precomposed form with `о`, so it stays in the
word after NFC, and `молоко́` and `молоко` are different words. In the second
row the hyphen follows the mark, so `молоко́-то` is one word. In `"a-\u0301b"`,
the stray mark after the hyphen is a separator, so the hyphen has no letter
after it and there are two words (if the mark were deleted first, the result
would be `a-b`). In `"1\u0301-b"`, the mark follows a digit, so it is not part
of any word, and the hyphen after it is a separator. In `"a\u0301\u0302"`, NFC
composes `a` + U+0301 into `á` (U+00E1); U+0302 has no composition with `á`, so
it stays in the word as a mark. U+20DD is an enclosing mark (`Me`), so it splits
`a` from `b`. The last row is Devanagari: `का` is
the letter KA (U+0915) followed by the spacing (`Mc`) vowel sign AA (U+093E).

### 4.6 No other text processing

- No stop words are removed. `the`, `и`, `мен` count like any other word.
- No stemming or lemmatization. `кот`, `кота`, and `коту` are three words.
- No markup is parsed. HTML tags, URLs, and e-mail addresses are just text and
  are split by the rules above.
- No minimum or maximum word length.

| Input                                      | Expected output                                                |
| ------------------------------------------ | -------------------------------------------------------------- |
| `"the cat and the dog and the bird"`       | `[("the", 3), ("and", 2), ("bird", 1), ("cat", 1), ("dog", 1)]` |
| `"кот кота коту кот"`                      | `[("кот", 2), ("кота", 1), ("коту", 1)]`                        |
| `"<b>news</b> news"`                       | `[("b", 2), ("news", 2)]`                                      |
| `"mail me@kbtu.kz"`                        | `[("kbtu", 1), ("kz", 1), ("mail", 1), ("me", 1)]`              |
| `"see https://kbtu.kz now"`                | `[("https", 1), ("kbtu", 1), ("kz", 1), ("now", 1), ("see", 1)]` |

## 5. Mixed scripts

A word may mix scripts; the rules in section 4 do not look at the script. A
Cyrillic word with a Latin look-alike letter in it is a different word from the
all-Cyrillic one.

| Input                            | Expected output                              |
| -------------------------------- | -------------------------------------------- |
| `"Pythonда pythonда"`            | `[("pythonда", 2)]`                          |
| `"\u043c\u0430\u043c\u0430 \u043c\u0061\u043c\u0430"` | `[("\u043c\u0061\u043c\u0430", 1), ("\u043c\u0430\u043c\u0430", 1)]` |

The second row is written with escapes so the letters can be told apart. Both
words look like `мама`, but in the second one the first `а` is Latin `a`
(U+0061) instead of Cyrillic `а` (U+0430), so they are different words. Because
U+0061 sorts before U+0430, the mixed word comes first (see section 6).

## 6. Ordering and ties

The result is sorted by count, highest first. Words with the same count are
sorted by Python's default string order (`str` comparison, which compares
Unicode code points), not by any language's alphabetical order. Letters below
U+0400 (Latin up to Latin Extended-B, which covers English and Kazakh Latin, plus
IPA, modifier letters, and Greek) sort before Cyrillic; some other Latin letters, such as Vietnamese `ạ`
(U+1EA1), sort after it. Order of appearance in the
text does not matter.

| Input                          | Expected output                                   |
| ------------------------------ | ------------------------------------------------- |
| `"zebra apple mango"`          | `[("apple", 1), ("mango", 1), ("zebra", 1)]`      |
| `"zoo \u00e4"`                  | `[("zoo", 1), ("\u00e4", 1)]`                     |
| `"ұл ул үй уй"`                | `[("уй", 1), ("ул", 1), ("үй", 1), ("ұл", 1)]`    |
| `"ёж жук"`                     | `[("жук", 1), ("ёж", 1)]`                         |
| `"\u1ea1 \u0431"`               | `[("\u0431", 1), ("\u1ea1", 1)]`                  |
| `"b a b a c"`                  | `[("a", 2), ("b", 2), ("c", 1)]`                  |

In the third row, `у` is U+0443, `ү` is U+04AF, and `ұ` is U+04B1, so every
word starting with `у` comes first, then `ү`, then `ұ`. Kazakh alphabetical
order is different (`у`, `ұ`, `ү`), so a collation-based sort would put `ұл`
before `үй`. Within the `у` words, `й` (U+0439) comes before `л` (U+043B). In
the fourth row, `ж` is U+0436 and `ё` is U+0451, so `жук` comes first, while
Russian alphabetical order (`е`, `ё`, `ж`) would put `ёж` first. In the second
row (`zoo ä`), `ä` is U+00E4, which sorts after `z` (U+007A); a collation-based
sort would put `ä` first, next to `a`. The fifth row is `ạ б`: `ạ` (U+1EA1) sorts after
`б` (U+0431).

## 7. The limit of 10

The result has `min(10, number of distinct words)` entries. If there are fewer
than 10 distinct words, all of them are returned. If there are more than 10
distinct words and several of them tie at the 10th place, the order from section
6 decides which ones are kept, and the result has exactly 10 entries.

| Input                                                     | Expected output |
| --------------------------------------------------------- | --------------- |
| `"a b c d e f g h i j k l"`                               | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` |
| `"l l k j i h g f e d c b a"`                             | `[("l", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` |
| `"k k j j a b c d e f g h i"`                             | `[("j", 2), ("k", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1)]` |

In the first row, `k` and `l` tie with the others at count 1 but are cut because
they sort last. In the second row, `l` appears last in the text but has the
highest count, so it is first; `j` and `k` are cut. In the third row, `j` and
`k` come first because of their count, even though `a` sorts before them; of the
nine words with count 1, `i` sorts last and is cut.

## 8. Out of scope

- Reading the article from a file or URL, or rendering the "Trending words" box.
- Choosing stop words or merging word forms; a later version may add these as
  separate options without changing the default behavior specified here.
- Performance tuning. The only requirement is O(n log n) time or better in the
  length of the text, for example one pass to count words with a `dict` or
  `Counter`, followed by `sorted` or `heapq.nsmallest` with the key from section
  6.
