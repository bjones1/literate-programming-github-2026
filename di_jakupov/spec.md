# Top words

A news site shows a "Trending words" box beside each article. This spec
defines a Python function that returns the 10 most common words in an
article's text and how many times each one appears. The text may be in
English, Russian or Kazakh (Cyrillic, or the 2021 Latin alphabet).

## Interface

Put the function in `top_words.py`. Use Python 3.9 or later and only the
standard library:

```python
def top_words(text: str) -> list[tuple[str, int]]:
```

- `text` is the whole article as plain text (markup already removed and
  entities decoded). If `isinstance(text, str)` is false (for example `None`
  or `bytes`), raise `TypeError`.
- Return a list of `(word, count)` tuples, at most 10 of them, sorted as
  described in rule 5. Return `[]` when the text has no words.

## Rules

1. Normalize the text in this order:
   1. Apply `unicodedata.normalize("NFC", ...)`, then `str.casefold()`.
   2. Delete U+00AD (soft hyphen), U+200C, U+200D, U+2060 and U+FEFF. These
      are invisible characters that can appear inside a word. U+200B (zero
      width space) is a space, so it is not deleted and separates words.
   3. Replace `’` (U+2019) and `ʼ` (U+02BC) with `'` (U+0027), and `‐`
      (U+2010) and `‑` (U+2011) with `-` (U+002D).
   4. First replace `ı` (U+0131) with `i`. Then replace `i` followed by U+0307
      (what `casefold()` makes from `İ`) with `i`. Each replacement in steps 2
      to 4 is one `str.replace()` call. This keeps Kazakh Latin words
      case-insensitive. The cost is that Kazakh Latin words that differ only
      in `ı` and `i`, like `ıs` and `is`, count as one word.
   5. Apply NFC again.

   All later rules work on this normalized text, and the returned words are
   in this form, so the box shows "ilim" for "Ilım".
2. A letter is a character whose `unicodedata.category()` starts with `L`. A
   mark is a character whose category starts with `M`. A digit is a
   character whose category is `Nd`. A joiner is `'` or `-`.
3. A candidate is a substring of the form `W (W|M)* ( J W (W|M)* )*`, where
   `W` is a letter or digit, `M` is a mark and `J` is one joiner. Scan the
   text from left to right and take non-overlapping candidates, each one as
   long as possible. A candidate is a word only if it contains at least one
   letter, so "covid-19", "5g" and "1-ші" are words but "2024" is not. Every
   character outside a word is a separator.
4. Count how many times each word occurs. Words are compared as exact strings.
   There is no stop-word list, no stemming and no merging of look-alike
   letters from different scripts (`е` and `ё`, Latin `a` and Cyrillic `а`,
   or a Latin `i` typed for the Kazakh `і`). Inflected forms like
   "қазақстан" and "қазақстанның" are different words. The task asks for the
   most common words, so function words like "the" and "в" will often lead
   the box, and this is accepted.
5. Sort the words by count, largest first. Break ties by the word in
   ascending Python string order (code point order, not dictionary order:
   `ё` and the Kazakh letters come after `я`). Then keep the first 10.

Letter and mark categories and case folding come from the Unicode tables of
the Python that runs the function. The examples below were checked on Python
3.13 (Unicode 15.1).

## Examples

Inputs are Python string literals.

| Decision | Input | Expected output |
| --- | --- | --- |
| Counting | `"cat dog cat bird dog cat"` | `[('cat', 3), ('dog', 2), ('bird', 1)]` |
| Case folding, not lower() | `"Cat CAT \ufb01nance FINANCE Қазақ ҚАЗАҚ"` | `[('cat', 2), ('finance', 2), ('қазақ', 2)]` |
| е and ё are different | `"ёж еж ёж"` | `[('ёж', 2), ('еж', 1)]` |
| Composed and decomposed forms match | `"\u0451ж \u0435\u0308ж"` | `[('ёж', 2)]` |
| Second NFC after deleting | `"\u0435\u00ad\u0308ж \u0451ж"` | `[('ёж', 2)]` |
| Kazakh Latin İ, I and ı | `"İnternet internet Ilım ılım ıs is"` | `[('ilim', 2), ('internet', 2), ('is', 2)]` |
| Punctuation separates | `"cat,dog.cat!(cat)"` | `[('cat', 3), ('dog', 1)]` |
| Apostrophes inside a word | `"don't Don\u2019t don\u02bct"` | `[("don't", 3)]` |
| Quotes around a word | `"'cat' \u2018cat\u2019 «кот»"` | `[('cat', 2), ('кот', 1)]` |
| Hyphen inside a word | `"кто-то state-of-the-art"` | `[('state-of-the-art', 1), ('кто-то', 1)]` |
| Unicode hyphens join too | `"кто\u2010то кто\u2011то"` | `[('кто-то', 2)]` |
| Hyphen at the edge or doubled | `"-cat- a--b"` | `[('a', 1), ('b', 1), ('cat', 1)]` |
| Other dashes separate | `"a\u2013b a\u2014b"` | `[('a', 2), ('b', 2)]` |
| Digits inside words, underscore separates | `"covid-19 mp3 5G 1-ші 2024 snake_case"` | `[('1-ші', 1), ('5g', 1), ('case', 1), ('covid-19', 1), ('mp3', 1), ('snake', 1)]` |
| Invisible characters are deleted | `"пере\u00adход переход a\u200cb a\u200db a\u2060b a\ufeffb"` | `[('ab', 4), ('переход', 2)]` |
| Zero width space separates | `"a\u200bb"` | `[('a', 1), ('b', 1)]` |
| Marks stay in the word | `"замо\u0301к замок"` | `[('замо\u0301к', 1), ('замок', 1)]` |
| Mark before a joiner | `"кто\u0301-то"` | `[('кто\u0301-то', 1)]` |
| Mark at word start separates | `"\u0301cat"` | `[('cat', 1)]` |
| Look-alike scripts | `"cat c\u0430t"` | `[('cat', 1), ('c\u0430t', 1)]` |
| Stop words, possessives, abbreviations | `"the Tokayev's Tokayev т.е. the"` | `[('the', 2), ('tokayev', 1), ("tokayev's", 1), ('е', 1), ('т', 1)]` |
| Ties in code point order | `"я b a Z ә"` | `[('a', 1), ('b', 1), ('z', 1), ('я', 1), ('ә', 1)]` |
| Cut after sorting | `"k j i h g f e d c b a k"` | `[('k', 2), ('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1)]` |
| No words | `""`, `"123 ... -- 2024-2025"` | `[]` |
| Not a string | `None`, `b"cat"` | raises `TypeError` |
