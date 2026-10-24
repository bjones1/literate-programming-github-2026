# Specification: `top_words`

## 1. Overview

`top_words(text: str) -> list[tuple[str, int]]` returns up to 10 most frequent words and their occurrence counts.

The function processes English, Russian, Kazakh, and mixed-language text. Its processing order is:

**Validation → NFC normalization → case folding → NFC normalization → tokenization → stopword filtering → counting → sorting → top 10.**

Results are deterministic for identical inputs under the same Python and Unicode database versions.

## 2. Input Validation

- Accept `str` and its subclasses.
- Raise `TypeError` for non-string inputs.
- Raise `ValueError` if the raw input contains any lone surrogate code points (U+D800–U+DFFF).
- Perform the type check before the surrogate check. Scan the raw input for surrogates before normalization.
- Obtain the plain string using `str.__str__(text)`, without invoking overridden subclass methods.
- Use unbound `str` methods, such as `str.casefold(value)`, for subsequent string operations where applicable.
- Return only plain `str` objects in the result, never instances of a `str` subclass.
- Do not impose undocumented input-size limits or produce external side effects.

Validation must precede all text processing.

## 3. Normalization

Normalize the input in this order:

1. Apply Unicode NFC normalization.
2. Apply Python's `str.casefold()`.
3. Apply Unicode NFC normalization again.

Use the Unicode database provided by Python. Results may differ between Unicode database versions.

The normalization procedure is applied to stopword entries as well. Each entry in the stopword lists must already be stored in the final normalized, case-folded form.

## 4. Tokenization

A letter is any Unicode character whose General Category starts with `L`. A mark is any Unicode character whose General Category starts with `M`, including `Mn`, `Mc`, and `Me`.

A letter group consists of one or more letters, optionally followed by marks attached to those letters. Marks cannot start a letter group. An unattached mark acts as a boundary and is discarded.

The only permitted internal separators are:

- ASCII hyphen-minus: `-` (U+002D)
- Hyphen: `‐` (U+2010)
- ASCII apostrophe: `'` (U+0027)
- Right single quotation mark: `’` (U+2019)

A token consists of one or more letter groups joined by single permitted separators. A separator is preserved only when it occurs between two complete letter groups, with exactly one permitted separator between them. Otherwise, it acts as a boundary and is discarded.

Digits, whitespace, punctuation, symbols, and control or format characters that are not permitted separators act as boundaries. Leading and trailing separators are discarded. Consecutive separators terminate the current token; they are not combined.

Do not unify separator variants. Tokens containing different permitted separator code points remain distinct. Stopword matching uses the exact normalized token spelling, including separator characters.

Preserve each token's normalized, case-folded spelling.

Examples:

| Input | Tokens |
|---|---|
| `state-of-the-art` | `state-of-the-art` |
| `hello--world` | `hello`, `world` |
| `can'’t` | `can`, `t` |
| `don't don’t` | `don't`, `don’t` |
| `e-mail e‐mail` | `e-mail`, `e‐mail` |
| `abc123def` | `abc`, `def` |
| `hello_world` | `hello`, `world` |
| `café` | `café` |
| `a-\u0301b` | `a`, `b` |
| `canʼt` | `canʼt` |
| `a\u00adb` | `a`, `b` |
| `a\u200cb a\u200db` | `a`, `b`, `a`, `b` |
| `e‑mail` | `e`, `mail` |
| `a−b` | `a`, `b` |
| `你好世界` | `你好世界` |

Here, `ʼ` is U+02BC, `­` is U+00AD, `‌` is U+200C, `‍` is U+200D, `‑` is U+2011, and `−` is U+2212.

Text written in scripts that normally do not separate words with spaces, such as Chinese, Japanese, and Thai, is not segmented linguistically. A continuous run of letters is treated as one token unless a character defined above creates a boundary.

## 5. Stopword Filtering

Remove tokens that exactly match an entry in the fixed, combined English, Russian, and Kazakh stopword lists below. All entries are stored in their normalized, case-folded form, using NFC normalization, Python case folding, and NFC normalization again, as specified in §3.

**English stopwords:**

```python
ENGLISH_STOPWORDS = {
    "a", "an", "and", "are", "as", "at", "be", "been", "being",
    "but", "by", "can", "could", "did", "do", "does", "doing",
    "for", "from", "had", "has", "have", "having", "he", "her",
    "here", "hers", "him", "his", "how", "i", "if", "in", "into",
    "is", "it", "its", "me", "more", "most", "my", "no", "nor",
    "not", "of", "on", "once", "only", "or", "other", "our",
    "ours", "out", "over", "own", "same", "she", "should", "so",
    "some", "such", "than", "that", "the", "their", "theirs",
    "them", "then", "there", "these", "they", "this", "those",
    "through", "to", "too", "under", "until", "up", "very", "was",
    "we", "were", "what", "when", "where", "which", "while", "who",
    "whom", "why", "will", "with", "would", "you", "your", "yours"
}
```

**Russian stopwords:**

```python
RUSSIAN_STOPWORDS = {
    "а", "без", "более", "бы", "был", "была", "были", "было",
    "быть", "в", "вам", "вас", "весь", "во", "вот", "все",
    "всё", "всего", "всех", "вы", "где", "да", "даже", "для",
    "до", "его", "ее", "её", "если", "есть", "ещё", "же",
    "за", "здесь", "и", "из", "или", "им", "их", "к", "как",
    "ко", "когда", "кто", "ли", "либо", "мне", "может", "мы",
    "на", "над", "надо", "наш", "не", "него", "нее", "неё",
    "нет", "ни", "них", "но", "ну", "о", "об", "однако",
    "он", "она", "они", "оно", "от", "по", "под", "при",
    "про", "с", "сам", "свою", "себе", "себя", "так", "также",
    "такой", "там", "те", "тем", "то", "того", "тоже", "той",
    "только", "том", "ты", "у", "уже", "чего", "чей", "чем",
    "что", "чтобы", "эта", "эти", "это", "я"
}
```

**Kazakh stopwords:**

```python
KAZAKH_STOPWORDS = {
    "ал", "ала", "алдында", "арқылы", "әр", "әрбір", "әркім",
    "әрқашан", "әртүрлі", "әрине", "әлі", "әдетте", "ба", "бе",
    "бен", "бері", "бірақ", "бір", "бірге", "бірде", "біреу",
    "бірнеше", "бірінші", "бірқатар", "бірқалыпты", "біріншіден",
    "бол", "болады", "болған", "болса", "болып", "болу", "бұл",
    "бұған", "бұдан", "бұрын", "бүкіл", "да", "де", "демек",
    "дейін", "деген", "дегенмен", "дәл", "екен", "емес", "енді",
    "үшін", "ғана", "ғой", "және", "жоқ", "жөн", "жоғары",
    "кейін", "кейбір", "керек", "кезде", "кім", "кімге", "кімнің",
    "көп", "ма", "ме", "мен", "мұнда", "мұны", "мұның", "мына",
    "мынадай", "мынау", "мың", "не", "немесе", "нәрсе", "осы",
    "осыған", "осындай", "осында", "осыдан", "осымен", "осының",
    "оған", "одан", "ол", "олар", "олардың", "оларға", "оларды",
    "онда", "ондай", "оның", "оң", "өте", "өз", "өзі",
    "өзін", "өзінің", "өзге", "өздері", "сайын", "себебі",
    "сияқты", "сол", "сонда", "сондай", "соң", "сонымен",
    "сондықтан", "тағы", "тек", "туралы", "ты", "у", "үш",
    "яғни", "и"
}
```

The combined stopword set is the union of these three sets. Matching is exact and language-independent. No language detection, stemming, lemmatization, substring matching, or additional stopword removal is permitted.

Separator characters are not interchangeable. For example, `don't` and `don’t` are distinct entries. A token is filtered only if its exact normalized spelling appears in the combined set.

## 6. Counting and Sorting

Count all occurrences of tokens remaining after stopword filtering.

Sort the distinct tokens by:

1. Frequency, descending.
2. Token string, ascending Unicode code-point order.

Use ordinary Python string ordering, not locale-aware sorting.

Return the first 10 `(word, count)` tuples. Return fewer than 10 if fewer than 10 distinct tokens remain. Ties at the tenth position are resolved by the same token-string ordering; do not return extra tokens to include all tied entries.

## 7. Reference Tests

The following examples define the expected behavior. Tests involving stopwords assume that the original lists have been restored as specified in §5.

```python
top_words("Cat dog cat. Собака кошка собака. Мысық ит мысық.")
# [('cat', 2), ('мысық', 2), ('собака', 2),
#  ('dog', 1), ('ит', 1), ('кошка', 1)]

top_words("The quick brown fox jumps over the lazy dog.")
# [('brown', 1), ('dog', 1), ('fox', 1), ('jumps', 1),
#  ('lazy', 1), ('over', 1), ('quick', 1)]

top_words("state-of-the-art AI model's performance")
# [('ai', 1), ("model's", 1), ('performance', 1),
#  ('state-of-the-art', 1)]

top_words("Straße STRASSE")
# [('strasse', 2)]

top_words("café cafe\u0301")
# [('café', 2)]

top_words("hello--world abc123def")
# [('abc', 1), ('def', 1), ('hello', 1), ('world', 1)]

top_words("banana apple cherry banana apple cherry")
# [('apple', 2), ('banana', 2), ('cherry', 2)]

top_words("")
# []

top_words("the and of мы и")
# []

top_words(123)
# Raises TypeError

top_words("\ud800")
# Raises ValueError
```

Additional required tests:

```python
class CustomStr(str):
    def __str__(self):
        raise AssertionError("overridden __str__ must not be called")

    def casefold(self):
        raise AssertionError("overridden casefold must not be called")

result = top_words(CustomStr("Cat cat"))
assert result == [("cat", 2)]
assert type(result[0][0]) is str
```

```python
top_words("a\u0301 á")
# [('á', 2)]

top_words("a-\u0301b")
# [('a', 1), ('b', 1)]

top_words("don't don’t e-mail e‐mail")
# [('don't', 1), ('don’t', 1),
#  ('e-mail', 1), ('e‐mail', 1)]
```

The following test verifies Unicode code-point ordering for equal frequencies:

```python
top_words("z é a")
# [('a', 1), ('z', 1), ('é', 1)]
```

The following test verifies the 10-token limit and tie-breaking at the cutoff:

```python
top_words("k j i h g f e d c b a")
# [('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1),
#  ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]
```

The following tests verify that distinct separator variants remain distinct:

```python
top_words("don't don’t")
# [("don't", 1), ("don’t", 1)]

top_words("e-mail e‐mail")
# [('e-mail', 1), ('e‐mail', 1)]
```

Stopword-related expected outputs must be verified against the restored original lists. Tests must also cover a string subclass that overrides other relevant methods, a raw surrogate in a subclass, more than 10 distinct tokens with unequal frequencies, and Unicode characters from each relevant General Category.

## 8. Implementation Contract

The implementation must follow this specification exactly, use no locale-dependent behavior, preserve the input, and return no more than 10 results.

Identical inputs must produce identical results under the same Python and Unicode database versions.