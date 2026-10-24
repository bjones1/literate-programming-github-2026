## Specification: `top_words`

**File:** `top_words.py`. **Python:** 3.9+, standard library only.  
**Signature:** `top_words(text: str) -> list[tuple[str, int]]`

Returns up to 10 `(word, count)` pairs for a plain-text article (no HTML parsing).

### Rules

1. **Preprocessing, in this order:** NFC-normalize; replace `’` (U+2019) and `ʼ` (U+02BC) with ASCII `'`; apply `str.casefold()`; NFC-normalize again. The returned word is this final form (`ß` becomes `ss`, `ς` becomes `σ`). `ё` is not merged with `е`, and no locale-specific casing is applied.
2. **Word characters:** a letter (`str.isalpha()`), a digit (`str.isdigit()`), or a combining mark (Unicode category `M*`) that directly follows a letter or another mark.
3. **Candidate token:** a maximal run of word characters. ASCII `'` or `-` is part of the token only when it sits between two word characters and neighbors no other joiner. Everything else is a separator: whitespace, punctuation, `_`, symbols, emoji, and other dashes (U+2010, U+2011, en/em dash).
4. **Digits:** a candidate token that contains any digit is discarded entirely (`abc123`, `2024`, `mp3` are not counted, and no `abc` is extracted).
5. **No stemming, no stop words.** `run` and `running` are different words, and `the`, `и`, `және` count like any other word.
6. **Scripts:** tokens are not split by script. Latin and Cyrillic letters mixed in one run form one word. Kazakh letters are ordinary letters. A run of CJK letters (no spaces) is one word.
7. **Single-letter words count.**
8. **Order:** count descending, then ties by ascending code-point order of the word (a prefix sorts before its extension).
9. **Result size:** at most 10 pairs, cut after sorting (so ties at the cutoff are resolved by rule 8). Fewer than 10 distinct words returns all of them. No words returns `[]`.
10. **Input type:** a `str` or `str` subclass is accepted. Any other type (including `bytes` and `None`) raises `TypeError`.

### Examples

|#|Decision|Input|Expected output|
|---|---|---|---|
|1|Basic counting|`cat dog cat bird dog cat`|`[("cat",3),("dog",2),("bird",1)]`|
|2|Case merged|`The the THE`|`[("the",3)]`|
|3|Cyrillic case|`Привет привет ПРИВЕТ`|`[("привет",3)]`|
|4|Punctuation|`Hi, hi! hi?`|`[("hi",3)]`|
|5|Inner apostrophe, curly = straight|`don't Don’t dont`|`[("don't",2),("dont",1)]`|
|6|Modifier apostrophe U+02BC|`donʼt don't`|`[("don't",2)]`|
|7|Edge apostrophe|`'cat' cat`|`[("cat",2)]`|
|8|Multiple apostrophes|`rock'n'roll`|`[("rock'n'roll",1)]`|
|9|Inner hyphen, tie order|`well-known well known`|`[("known",1),("well",1),("well-known",1)]`|
|10|Edge hyphen / spaced dashes|`cat - cat -- dog`|`[("cat",2),("dog",1)]`|
|11|Consecutive joiners split|`a--b a-'b`|`[("a",2),("b",2)]`|
|12|Other dashes are separators|`well‐known a–b`|`[("a",1),("b",1),("known",1),("well",1)]`|
|13|Underscore separates|`snake_case`|`[("case",1),("snake",1)]`|
|14|Digit tokens dropped|`2024 abc123 cat`|`[("cat",1)]`|
|15|Dropped, not truncated|`abc123 abc`|`[("abc",1)]`|
|16|Kazakh letters|`Қазақстан қазақстан Ұлы`|`[("қазақстан",2),("ұлы",1)]`|
|17|`ё` ≠ `е`, code-point tie order|`ёж еж`|`[("еж",1),("ёж",1)]`|
|18|Casefold expansion|`Straße STRASSE`|`[("strasse",2)]`|
|19|NFC (decomposed + precomposed é)|`e\u0301 e\u0301 é`|`[("é",3)]`|
|20|Stress mark kept in word|`Москва\u0301 москва`|`[("москва",1),("москва\u0301",1)]`|
|21|Leading lone mark ignored|`\u0301cat`|`[("cat",1)]`|
|22|No stemming|`run running runs`|`[("run",1),("running",1),("runs",1)]`|
|23|Stop words kept|`и и и дом`|`[("и",3),("дом",1)]`|
|24|Mixed-script tokens|`привет hello привет`|`[("привет",2),("hello",1)]`|
|25|Latin+Cyrillic in one token|`aб aб` (Latin `a`)|`[("aб",2)]`|
|26|CJK run is one word|`東京 東京`|`[("東京",2)]`|
|27|Single letters|`я a я`|`[("я",2),("a",1)]`|
|28|Whitespace types|`cat\ncat\t cat`|`[("cat",3)]`|
|29|Tie-break alphabetical|`b a c a b c`|`[("a",2),("b",2),("c",2)]`|
|30|Top-10 cutoff|`a`×11, `b`×10, … `j`×2, `k`×1|`[("a",11),("b",10),…,("j",2)]`; `k` dropped|
|31|Tie at cutoff|`a`–`i` ×2 each, plus `bbb` and `aaa` once|`a`–`i` with 2, then `("aaa",1)`; `bbb` dropped|
|32|Fewer than 10|`one two`|`[("one",1),("two",1)]`|
|33|Empty / no words|`""`, `"123 !!!"`|`[]`|
|34|`str` subclass accepted|`class S(str): pass`, `S("a a")`|`[("a",2)]`|
|35|Bad type|`None`, `b"cat"`|raises `TypeError`|