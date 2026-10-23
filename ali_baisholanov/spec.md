# Top words specification

## Purpose and interface

Implement `top_words(text)` in `top_words.py` using Python 3. It returns the
10 most frequent words in an article, with their occurrence counts. The text
may contain English, Russian, Kazakh, or a mixture of languages.

The input must be a Python `str`. For every other input type, raise `TypeError`;
the exception message is unspecified. Do not coerce values to strings. The
function accepts the article text itself, not a filename or URL, and performs
no file access, network access, printing, or interactive input.

Return a `list` of `(word, count)` tuples, where `word` is a `str` and `count`
is a positive `int`. Return `[]` if there are no words. If there are fewer than
10 distinct words, return all of them; otherwise return exactly 10. The limit
is fixed, with no additional arguments.

## Word recognition and counting

Apply these steps in this order:

1. Normalize the complete input with `unicodedata.normalize("NFC", text)`.
2. Apply Python's `str.casefold()` to the complete normalized string.
3. Normalize the result to NFC again.
4. Split that result into maximal, nonempty, consecutive runs of characters
   for which Python's `str.isalpha()` returns `True`. Each run is one word.
   Every other character is a separator; separators are never deleted in a
   way that joins the words on their two sides.
5. Count every occurrence of each resulting word across the complete text.

These definitions use the Unicode character data supplied by the Python
runtime, without locale-specific rules. NFC makes canonically equivalent
spellings equal; compatibility normalization (NFKC) is not performed.
Case folding is applied before splitting, including any expansions it makes.
Combining marks that remain after the transformations are separators because
they do not satisfy `isalpha()`.

Letters from every script are accepted. Adjacent letters from different
scripts form one word. Digits, underscores, punctuation, apostrophes, hyphens,
emoji, and whitespace are separators. Straight and curly apostrophes behave
identically. Treat markup as literal text: do not interpret HTML or Markdown.

Count short words and common words (including articles and conjunctions).
Do not remove stop words, stem words, translate text, or replace distinct
letters such as Russian `е` and `ё`, or Kazakh `к` and `қ`.

## Ranking

Sort distinct words by decreasing count. When counts are equal, sort their
transformed word strings in ascending Python string order (lexicographic
Unicode code point order, with a shorter prefix preceding its extension).
Apply this ordering before taking the first 10 entries. Do not include extra
entries tied with the tenth word. Input order and locale do not break ties.

## Acceptance examples

Each input cell is a Python literal. Escape sequences represent actual
characters; for example, `\n` is a newline and `\u0301` is a combining acute
accent. Expected results are exact Python values. A `TypeError` entry means
the call must raise that exception. These cases are independent calls.

| Decision exercised | Input | Expected result |
| --- | --- | --- |
| Count repeated words; return fewer than 10 | `'cat dog cat bird dog cat'` | `[('cat', 3), ('dog', 2), ('bird', 1)]` |
| Empty text | `''` | `[]` |
| Whitespace only | `' \t\n\r '` | `[]` |
| Text with no letters | `'123 _ -- 😀 !'` | `[]` |
| English case folding | `'Cat CAT cat'` | `[('cat', 3)]` |
| Russian case folding; retain short words | `'Мир МИР и мир'` | `[('мир', 3), ('и', 1)]` |
| Kazakh letters and case folding | `'ҚАЗАҚ қазақ ӘҢ ҮҰ ӨІ Һ'` | `[('қазақ', 2), ('үұ', 1), ('һ', 1), ('әң', 1), ('өі', 1)]` |
| Preserve distinct Russian letters | `'елка ёлка ЕЛКА'` | `[('елка', 2), ('ёлка', 1)]` |
| Preserve distinct Kazakh letters | `'қ к Қ'` | `[('қ', 2), ('к', 1)]` |
| Punctuation separates instead of joining | `'cat,dog.cat!dog?'` | `[('cat', 2), ('dog', 2)]` |
| Apostrophes split words | `"don't don’t"` | `[('don', 2), ('t', 2)]` |
| Hyphens and dashes split words | `'well-being well—being'` | `[('being', 2), ('well', 2)]` |
| Digits and underscores split words | `'abc123def abc_def 42'` | `[('abc', 2), ('def', 2)]` |
| Newlines, tabs, and nonbreaking spaces separate | `'a\nb\ta\u00a0b'` | `[('a', 2), ('b', 2)]` |
| Emoji separates words | `'hi😀hi'` | `[('hi', 2)]` |
| NFC combines canonically equivalent spellings | `'café cafe\u0301 CAFÉ'` | `[('café', 3)]` |
| Retain accents | `'cafe café'` | `[('cafe', 1), ('café', 1)]` |
| Case folding may expand a letter | `'Straße STRASSE'` | `[('strasse', 2)]` |
| Marks remaining after NFC separate words | `'x\u0301y'` | `[('x', 1), ('y', 1)]` |
| Transform before splitting; case folding introduces a mark | `'İX'` | `[('i', 1), ('x', 1)]` |
| Do not apply compatibility normalization | `'Ａ A'` | `[('a', 1), ('ａ', 1)]` |
| Accept other scripts | `'猫 猫 dog'` | `[('猫', 2), ('dog', 1)]` |
| Adjacent scripts stay in one word | `'catкот cat кот'` | `[('cat', 1), ('catкот', 1), ('кот', 1)]` |
| Do not strip markup | `'<b>Cat</b>'` | `[('b', 2), ('cat', 1)]` |
| No stop-word filtering or stemming | `'the and the cats cat'` | `[('the', 2), ('and', 1), ('cat', 1), ('cats', 1)]` |
| Frequency takes priority over alphabetic order | `'z z b a'` | `[('z', 2), ('a', 1), ('b', 1)]` |
| Ties ignore first occurrence; prefix order | `'aa a ab'` | `[('a', 1), ('aa', 1), ('ab', 1)]` |
| Ties use code point order across scripts | `'я қ a ә'` | `[('a', 1), ('я', 1), ('қ', 1), ('ә', 1)]` |
| Exactly 10 distinct words | `'j i h g f e d c b a'` | `[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` |
| More than 10; resolve a tie at the cutoff | `'k j i h g f e d c b a a'` | `[('a', 2), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]` |
| Reject missing text | `None` | `TypeError` |
| Reject numbers | `123` | `TypeError` |
| Reject encoded bytes | `b'cat'` | `TypeError` |
| Reject collections | `['cat']` | `TypeError` |

## Scope

This is a specification of a function, not a command-line program or web UI.
The chosen tokenization is deliberately mechanical: contractions and
hyphenated expressions split as specified above. Language-specific linguistic
analysis is outside the scope. No particular counting algorithm, exact error
message, or performance benchmark is required.

## Source

Based on the [Top words exercise](https://github.com/bjones1/literate-programming-book/blob/main/exercises/top-words.md).
