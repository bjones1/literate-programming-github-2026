`top_words` specification
=========================

Interface
---------

* `top_words(text)` lives in `top_words.py` and is written in Python 3.
* `text` must be a `str` (subclasses of `str` are accepted). Any other type
  (`None`, numbers, `bool`, `bytes`, `bytearray`, lists, and so on) raises
  `TypeError`. Values are never converted to strings.
* The function takes the article text itself, not a filename or URL. It does no
  file access, network access, printing or input.
* It returns a `list` of `(word, count)` tuples, with `word` a `str` and `count`
  a positive `int`.
* The limit of 10 is fixed, with no extra parameters.
* If there are no words, it returns `[]`. If there are fewer than 10 distinct
  words, it returns all of them. Otherwise it returns exactly 10.

What counts as a word
---------------------

Apply these steps in order:

1. Run `text.casefold()` on the whole string, then normalize with
   `unicodedata.normalize("NFC", ...)`.
2. Replace every curly apostrophe `’` (U+2019) with a straight apostrophe `'`.
3. A **letter** is any character where `str.isalpha()` is `True`, in any script.
4. A **word** is a maximal run of letters. Inside a run, a single apostrophe may
   join two letters, but only if a letter sits directly on both sides of it. The
   word keeps the apostrophe as a straight `'`.
5. Every other character is a separator that never joins its neighbours. This
   includes digits, underscores, hyphens, dashes, other punctuation, whitespace,
   emoji, leftover combining marks, and apostrophes that don't meet the rule in
   step 4 (leading, trailing, or doubled).

Only `'` and `’` count as apostrophes. Lookalikes follow the normal rules:
`‘` (U+2018) is punctuation and therefore a separator, while `ʼ` (U+02BC) passes
`isalpha()` and is therefore a letter.

What the spec deliberately does **not** do:

* No stop-word removal, stemming, translation or compatibility normalization
  (NFKC).
* No markup interpretation. HTML and Markdown are plain text.
* No merging of distinct letters. `е`/`ё`, `к`/`қ`, and accented versus plain
  Latin letters stay different.
* No word segmentation for Chinese or Japanese. A run of letters is one word.
* Mixed-script runs such as `catкот` are a single word.

Ranking
-------

1. Sort by count, highest first.
2. Break ties by the word string in ascending code point order (Python's default
   `str` comparison).
3. Take the first 10 entries. Words tied with the 10th that fall after the
   cutoff are dropped.

Examples
--------

Each row is an independent call and the expected value is exact. The last column
names the decision the row pins down.

| Input (Python literal)               | Expected result                                                                                        | Decision                                                  |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| `'the cat saw the dog and the bird'` | `[('the', 3), ('and', 1), ('bird', 1), ('cat', 1), ('dog', 1), ('saw', 1)]`                            | Counting, fewer than 10, no stop-word removal             |
| `''`                                 | `[]`                                                                                                   | Empty text                                                |
| `' \n\t '`                           | `[]`                                                                                                   | Whitespace only                                           |
| `'2024 -- !!! 😀'`                   | `[]`                                                                                                   | No letters                                                |
| `'Apple APPLE apple'`                | `[('apple', 3)]`                                                                                       | Case folding                                              |
| `'Привет ПРИВЕТ мир'`                | `[('привет', 2), ('мир', 1)]`                                                                          | Russian case folding                                      |
| `'Қала қала ҚАЛА Әке'`               | `[('қала', 3), ('әке', 1)]`                                                                            | Kazakh letters and case folding                           |
| `'ёж еж ЁЖ'`                         | `[('ёж', 2), ('еж', 1)]`                                                                               | `ё` and `е` stay distinct                                 |
| `'қыс кыс ҚЫС'`                      | `[('қыс', 2), ('кыс', 1)]`                                                                             | `қ` and `к` stay distinct                                 |
| `"don't Don’t dont"`                 | `[("don't", 2), ('dont', 1)]`                                                                          | Inner apostrophes stay, curly equals straight             |
| `"don‘t"`                            | `[('don', 1), ('t', 1)]`                                                                               | Only `'` and `’` are apostrophes                          |
| `"'hello' rock 'n' roll"`            | `[('hello', 1), ('n', 1), ('rock', 1), ('roll', 1)]`                                                   | Edge apostrophes are separators                           |
| `"a''b"`                             | `[('a', 1), ('b', 1)]`                                                                                 | Doubled apostrophe doesn't join                           |
| `'state-of-the-art'`                 | `[('art', 1), ('of', 1), ('state', 1), ('the', 1)]`                                                    | Hyphens split                                             |
| `'abc123def abc_def'`                | `[('abc', 2), ('def', 2)]`                                                                             | Digits and underscores split                              |
| `'hi😀hi'`                           | `[('hi', 2)]`                                                                                          | Emoji separates                                           |
| `'cafe\u0301 café'`                  | `[('café', 2)]`                                                                                        | NFC unifies equivalent spellings                          |
| `'naive naïve'`                      | `[('naive', 1), ('naïve', 1)]`                                                                         | Accents are kept                                          |
| `'Straße STRASSE'`                   | `[('strasse', 2)]`                                                                                     | Casefold expansion (not just `lower()`)                   |
| `'x\u0301y'`                         | `[('x', 1), ('y', 1)]`                                                                                 | Leftover combining mark is a separator                    |
| `'Ａ A'`                             | `[('a', 1), ('ａ', 1)]`                                                                                | No NFKC                                                   |
| `'猫 猫 dog'`                        | `[('猫', 2), ('dog', 1)]`                                                                              | Other scripts count as letters                            |
| `'catкот cat кот'`                   | `[('cat', 1), ('catкот', 1), ('кот', 1)]`                                                              | Mixed scripts form one word                               |
| `'<p>Hi</p>'`                        | `[('p', 2), ('hi', 1)]`                                                                                | Markup is literal text                                    |
| `'z z b a'`                          | `[('z', 2), ('a', 1), ('b', 1)]`                                                                       | Count beats alphabetical order                            |
| `'b a b a c'`                        | `[('a', 2), ('b', 2), ('c', 1)]`                                                                       | Equal counts sort alphabetically, not by first appearance |
| `'я қ ә a'`                          | `[('a', 1), ('я', 1), ('қ', 1), ('ә', 1)]`                                                             | Ties use code point order across scripts                  |
| `'j i h g f e d c b a'`              | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | Exactly 10 distinct words                                 |
| `'m l k j i h g f e d c b a b'`      | `[('b', 2), ('a', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` | More than 10 words, tie cut at the limit                  |
| `None`                               | `TypeError`                                                                                            | Reject missing input                                      |
| `42`                                 | `TypeError`                                                                                            | Reject numbers                                            |
| `True`                               | `TypeError`                                                                                            | Reject `bool`                                             |
| `b'cat'`                             | `TypeError`                                                                                            | Reject bytes                                              |
| `['cat']`                            | `TypeError`                                                                                            | Reject collections                                        |