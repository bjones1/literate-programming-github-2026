# Top words — specification

A news site shows a "Trending words" box beside each article. This spec defines
exactly what that box contains, so that only one implementation is plausible.

## Interface

- Python 3.10 or later, standard library only.
- File `top_words.py` defines `top_words(text: str) -> list[tuple[str, int]]`.
- If `text` is not a `str`, raise `TypeError`. The function never reads files,
  prints, or mutates global state.
- Returns a list of at most 10 `(word, count)` tuples, where `word` is a `str`
  and `count` is an `int` ≥ 1.

| Input                      | Expected output                              |
| -------------------------- | -------------------------------------------- |
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]`    |
| `""`                       | `[]`                                         |
| `"   ...!!! 123 "`         | `[]`                                         |
| `None`                     | raises `TypeError`                           |
| `b"cat"`                   | raises `TypeError`                           |

## Processing steps

The function applies these steps, in this order.

### 1. Unicode normalization

Normalize `text` to NFC (`unicodedata.normalize("NFC", text)`), so a letter
typed as a base letter plus a combining mark equals its precomposed form. Then
delete every remaining character whose Unicode category is `Mn` (nonspacing
mark), such as the acute stress mark U+0301 used in Russian teaching texts.

| Input                                          | Expected output     |
| ---------------------------------------------- | ------------------- |
| `"й й"` where the second `й` is `и` + U+0306   | `[("й", 2)]`        |
| `"ма́ма мама"` (first word has U+0301 after `а`) | `[("мама", 2)]`   |

### 2. Apostrophe and hyphen normalization

Replace each of these characters with the ASCII apostrophe `'` (U+0027): right
single quotation mark `’` (U+2019) and modifier letter apostrophe `ʼ` (U+02BC).
Replace each of these with the ASCII hyphen-minus `-` (U+002D): hyphen `‐`
(U+2010) and non-breaking hyphen `‑` (U+2011). No other characters are
substituted; in particular the left single quotation mark `‘` (U+2018), en
dash, and em dash are left alone and act as separators.

| Input                                  | Expected output        |
| -------------------------------------- | ---------------------- |
| `"don’t don't donʼt"`                  | `[("don't", 3)]`       |
| `"e‑mail e-mail"` (first uses U+2011) | `[("e-mail", 2)]`      |
| `"‘tis"`                               | `[("tis", 1)]`         |

### 3. Splitting into words

A **letter** is any character `c` for which `c.isalpha()` is `True`. This
covers English, Russian, and Kazakh in both Cyrillic (`ә ғ қ ң ө ұ ү һ і`) and
Latin script.

A **word** is a maximal run of letters, where a single ASCII apostrophe `'` or
a single ASCII hyphen-minus `-` is also part of the word when it has a letter
immediately on **both** sides. Equivalently, a word matches the regular
expression `L+(?:[-']L+)*`, where `L` is a letter.

Everything else separates words, including whitespace, digits, punctuation,
underscores, en dash `–`, em dash `—`, and quotation marks. Digits are never
part of a word, and a number on its own is not a word.

| Input                          | Expected output                                      | Decision shown                     |
| ------------------------------ | ---------------------------------------------------- | ---------------------------------- |
| `"well-known well known"`      | `[("known", 1), ("well", 1), ("well-known", 1)]`     | internal hyphen joins              |
| `"don't dont"`                 | `[("don't", 1), ("dont", 1)]`                        | internal apostrophe joins          |
| `"'quoted' -dash- a--b"`       | `[("a", 1), ("b", 1), ("dash", 1), ("quoted", 1)]`   | edge / doubled marks separate      |
| `"COVID-19 2026 r2d2"`         | `[("covid", 1), ("d", 1), ("r", 1)]`                 | digits separate, never count       |
| `"word—word – word_word"`      | `[("word", 4)]`                                      | dashes and `_` separate            |
| `"Алматы-ға қала"`             | `[("алматы-ға", 1), ("қала", 1)]`                    | Kazakh hyphenated suffix joins     |
| `"x² x ½"`                     | `[("x", 2)]`                                         | superscripts, fractions separate   |

### 4. Case folding

Convert each word with `str.lower()`, then delete any `Mn` characters that
lowering produced (for example, `"İ".lower()` is `i` + U+0307, which becomes
plain `i`). No other folding is done: Russian `ё` and `е` stay distinct, no
accents are removed, and lowering is locale-independent, so Latin `I` always
becomes `i` (never Kazakh/Turkish dotless `ı`).

| Input                 | Expected output             |
| --------------------- | --------------------------- |
| `"İnternet internet"` | `[("internet", 2)]`         |
| `"Iı"`                | `[("iı", 1)]`               |
| `"The the THE"`       | `[("the", 3)]`              |
| `"Ёж ёж еж"`          | `[("ёж", 2), ("еж", 1)]`    |
| `"Қала ҚАЛА қала"`    | `[("қала", 3)]`             |

### 5. Counting

Count every word. There is **no** stop-word list and **no** minimum length:
words like `the`, `и`, `a`, `мен` count like any other word. Words in
different languages or scripts are always different words, even if they look
alike (Latin `c` ≠ Cyrillic `с`).

| Input                    | Expected output                  |
| ------------------------ | -------------------------------- |
| `"a a и и и the"`        | `[("и", 3), ("a", 2), ("the", 1)]` |

### 6. Ranking

Sort words by count, highest first. Break ties by the word in ascending
Python string order (Unicode code point order), so Latin-script words come
before Cyrillic words with the same count. Equivalently:
`sorted(counts.items(), key=lambda wc: (-wc[1], wc[0]))`.

| Input                         | Expected output                                   |
| ----------------------------- | ------------------------------------------------- |
| `"b a c"`                     | `[("a", 1), ("b", 1), ("c", 1)]`                  |
| `"мир peace мир peace"`       | `[("peace", 2), ("мир", 2)]`                      |

### 7. Truncation

Return the first 10 entries of the ranked list. If there are fewer than 10
distinct words, return all of them. A word tied with the 10th entry but
ranked 11th or later by the tie-break is **not** included.

| Input                                       | Expected output                                                                                                         |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `"k j i h g f e d c b a"` (11 distinct, all count 1) | `[("a",1),("b",1),("c",1),("d",1),("e",1),("f",1),("g",1),("h",1),("i",1),("j",1)]` — `k` is dropped |
| `"x x y"`                                   | `[("x", 2), ("y", 1)]`                                                                                                  |
