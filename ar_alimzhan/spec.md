# Top words — specification

A news site shows a "Trending words" box beside each article. This spec defines a Python function that returns the 10 most common words in an article's text, with the number of times each appears. Articles may be in English, Russian or Kazakh (Cyrillic or Latin script).

## 1. Interface

- File: `top_words.py`. Python 3.10 or newer, standard library only.
- Signature: `def top_words(text: str) -> list[tuple[str, int]]:`
- The function is pure: it does no I/O, does not mutate its argument, keeps no state between calls, and returns the same result for the same input.

## 2. Input

1. If `text` is not an instance of `str` (for example `bytes`, `None`, a list), raise `TypeError` with the message `text must be a str`. Instances of `str` subclasses are accepted and treated as plain strings: the function works on the underlying characters and must not call methods a subclass may override (`casefold`, `__iter__`, …). No conversion (decoding, `str()`) is attempted.
2. Any `str` is valid input, including the empty string, strings containing only punctuation, control characters, emoji or lone surrogates. None of these raise; they simply contain no words or act as separators.

## 3. Processing steps (in this order)

1. **Normalize** the whole input to Unicode NFC (`unicodedata.normalize("NFC", text)`) before anything else, so that a precomposed letter (`é`, U+00E9) and its decomposed form (`e` + U+0301) are the same text.
2. **Tokenize** the normalized text into words (section 4).
3. **Fold** each word into its counting key: `unicodedata.normalize("NFC", word.casefold())`, after replacing every U+2019 RIGHT SINGLE QUOTATION MARK in the word with U+0027 APOSTROPHE (section 4.3). A key is never split again after folding, even if folding changes its length (`Straße` becomes the single key `strasse`).
4. **Count** how many times each key occurs.
5. **Rank** and **truncate** (section 5).

## 4. What a word is

Characters are Unicode code points (not bytes and not grapheme clusters). Character classes use `unicodedata.category()` of the Python version running the code.

### 4.1 Character classes

| Class | Unicode general categories | Role |
|---|---|---|
| Letter | `Lu`, `Ll`, `Lt`, `Lm`, `Lo` | Can start, continue and end a word |
| Mark | `Mn`, `Mc`, `Me`, except variation selectors | Continues a word only if it directly follows a character that is already part of the word; otherwise it is a separator |
| Joiner | U+0027 `'`, U+2019 `’`, U+002D `-`, U+2010 `‐` | Part of a word only when a word is in progress (the previous character was kept as part of a word) **and** the next character of the NFC text is a Letter; otherwise a separator |
| Separator | everything else | Ends the current word and is discarded |

Notes on the class table:

- **Digits are separators** (`Nd`, `Nl`, `No`). `COVID-19` gives the word `covid`; `2026` gives no word; `abc2def` gives `abc` and `def`. Numbers are not "words" in a trending box.
- The underscore `_`, all other punctuation, symbols, emoji, whitespace of every kind (including U+00A0 NO-BREAK SPACE and line breaks), format characters (`Cf`, e.g. U+200B ZERO WIDTH SPACE, U+00AD SOFT HYPHEN) and variation selectors are separators. Variation selectors (U+FE00–U+FE0F and U+E0100–U+E01EF) have category `Mn`, but they are explicitly excluded from the Mark class, so `a\uFE0Fb` gives `a` and `b`. Likewise `co\u00ADop` is the two words `co` and `op`.
- The rule depends only on the category, not on how the character looks or is used in a language. For example U+02BC MODIFIER LETTER APOSTROPHE is category `Lm`, so it is a Letter.
- Lone surrogates (category `Cs`) are separators: `"ab\udc00cd"` gives `ab` and `cd`.

### 4.2 Word

A **word** is a maximal run of characters that starts with a Letter and continues with Letters, Marks and Joiners, following the rules above. Equivalently, scanning left to right:

- a Letter starts a word or extends the current one;
- a Mark extends the current word, or is a separator if no word is in progress;
- a Joiner extends the current word only if a word is in progress and the next character is a Letter; otherwise it ends the current word (if any) and is discarded;
- anything else ends the current word.

Consequences:

- `well-known` and `don't` are single words; `что-то`, `бала-шаға`, `rock'n'roll` are single words.
- `--word--`, `'quoted'`, `end-` give `word`, `quoted`, `end`: a leading or trailing joiner is not part of the word.
- `a--b` and `a'-b` give `a` and `b` (two joiners in a row: the first is not followed by a letter).
- `x-2` gives `x` (the hyphen is not followed by a letter).
- An em dash `—` (U+2014) or en dash `–` (U+2013) is a separator: `мир—труд` gives `мир` and `труд`.
- Kazakh is supported in Cyrillic and in the 2021 Latin alphabet (letters with diacritics such as `ş`, `ä`). The older 2018 Latin variant that writes letters as a base letter plus a trailing apostrophe (`s'`) is out of scope: a trailing apostrophe is a separator, so `a'` gives `a`.
- A possessive is part of the word: `Kazakhstan's` and `Kazakhstan` are two different keys. This is intentional: the function does no stemming or lemmatization.

### 4.3 Counting key

- The key is the folded word from step 3. Case folding uses `str.casefold()`, so `The`, `THE` and `the` count together, and `Кот`/`кот`, `Үй`/`үй`, `Ғылым`/`ғылым` count together.
- `’` and `'` are treated as the same character (`don’t` and `don't` are one key, written with `'`).
- Case folding is locale-independent: Turkish/Azerbaijani rules are not applied, so `İ` (U+0130) folds to `i` + U+0307 and `İstanbul` and `istanbul` are different keys.
- No other merging is done: `е` and `ё` are different keys, Kazakh `і` (U+0456) and Latin `i` (U+0069) are different keys, `color` and `colour` are different keys. Text in different scripts is never transliterated.
- No stopwords are removed. Words like `the`, `и`, `және` are counted like any other word. Filtering stopwords is the caller's job.
- The returned word is the key, not the original spelling.

## 5. Result

1. Sort the keys by count, highest first. Break ties by the key in ascending code-point order (Python's default `str` comparison). This makes the result fully deterministic and independent of the order in which words appear.
2. Return a new `list` of at most 10 tuples `(key, count)`, where `key` is a `str` and `count` is an `int` ≥ 1.
3. If there are fewer than 10 distinct keys, return all of them. If the text has no words, return `[]`.

## 6. Non-functional requirements

- Time O(n + k log k) for n characters and k distinct keys; an article of 1,000,000 characters must be processed in under 1 second on a typical laptop.
- Memory is proportional to the number of distinct keys plus the length of the input.

## 7. Acceptance tests

Each row is a required pytest case. Inputs are Python string literals, so escapes such as `\u0301` and `\t` stand for the single character they denote. The expected column is the exact return value (compared with `==`).

| # | Input | Expected result | Decision tested |
|---|---|---|---|
| 1 | `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | Counting and ranking |
| 2 | `"The the THE"` | `[("the", 3)]` | Case folding |
| 3 | `"Кот кот КОТ собака"` | `[("кот", 3), ("собака", 1)]` | Russian |
| 4 | `"Үй үй мектеп Ғылым ғылым"` | `[("ғылым", 2), ("үй", 2), ("мектеп", 1)]` | Kazakh letters; tie broken by code point |
| 5 | `"e\u0301cole école"` | `[("école", 2)]` | NFC before tokenizing |
| 6 | `"Straße STRASSE strasse"` | `[("strasse", 3)]` | Fold, then no re-splitting |
| 7 | `"don't don’t dont"` | `[("don't", 2), ("dont", 1)]` | Apostrophe joins; `’` equals `'` |
| 8 | `"well-known -well- что-то"` | `[("well", 1), ("well-known", 1), ("что-то", 1)]` | Inner hyphen joins; outer ones separate |
| 9 | `"COVID-19 2026 abc2def"` | `[("abc", 1), ("covid", 1), ("def", 1)]` | Digits are separators |
| 10 | `"мир—труд, май!"` | `[("май", 1), ("мир", 1), ("труд", 1)]` | Dashes and punctuation separate |
| 11 | `""`, `"123 !!! 😀"`, `"\u0301"` | `[]` | No words; leading mark is not a word |
| 12 | `"a\u00A0b\tc\nd\u200be"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1)]` | Unicode whitespace and format chars separate |
| 13 | `"b a"` and `"a b"` | both `[("a", 1), ("b", 1)]` | Result does not depend on word order |
| 14 | `"al ak aj ai ah ag af ae ad ac ab aa"` (12 distinct words, each once) | `[("aa", 1), ("ab", 1), ("ac", 1), ("ad", 1), ("ae", 1), ("af", 1), ("ag", 1), ("ah", 1), ("ai", 1), ("aj", 1)]` | Truncation to 10; ties by code point |
| 15 | `None`, `b"text"`, `42` | `TypeError("text must be a str")` | Input validation |
| 16 | `"Kazakhstan's Kazakhstan"` | `[("kazakhstan", 1), ("kazakhstan's", 1)]` | No stemming |
| 17 | `"Ё ё е"` | `[("ё", 2), ("е", 1)]` | `ё` and `е` are not merged |
| 18 | `"a\uFE0Fb a"` | `[("a", 2), ("b", 1)]` | Variation selector is a separator |

Under other plausible readings these tests give different results: a version that splits on apostrophes would return `("don", …)`, one that keeps digits would return `("2026", 1)`, one that removes stopwords would drop `the`, and one that sorts ties by first appearance would fail test 13.
