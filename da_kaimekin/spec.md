Intro

A news site shows a "Trending words" box beside each article. Write a Python (python 3.11.6) function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh.

This spec consists of multiple parts. Implement the following:
```def top_words(text: str, *, max_length: int = 1_000_000) -> str:
    return format_frequency_list(
        top_k_frequent(normalize(validate_article_text(text, max_length=max_length)), 10)
    )
```

Testing and validation

This section is normative: an implementation is accepted if and only if it passes every test below. Each test is grouped by the component under test and includes the exact inputs and expected outputs taken from the spec; where two components interact, the end-to-end `top_words` tests in §9 are authoritative.

---

## 1. Purpose and scope

The suite has three jobs:

1. **Unit-level correctness** — every component does what its section of the spec says, on every documented vector and every documented edge case.
    
2. **Contract-level correctness** — the pipeline `validate → normalize → top_k_frequent → format_frequency_list` produces the exact string in §9's table for every input.
    
3. **Invariant-level correctness** — properties (§7 of the error-handling section; §5 of the normalization section) hold under randomized input, not just the hand-written vectors.
    

The suite **does not** test: HTTP status mapping, logging, batch-job behavior, or code style. Those are integration/ops concerns (§10 of the error-handling section).

---

## 2. Test file layout

The reference implementation is a single module `top_words.py`. The test suite mirrors it:
```
text

tests/
├── conftest.py             # shared fixtures (§4)
├── test_normalize.py       # §5
├── test_validate.py        # §6
├── test_top_k.py           # §7
├── test_format.py          # §8
├── test_top_words.py       # §9 (end-to-end; the acceptance gate)
└── test_invariants.py      # §10 (property-based)
```

`pytest` with `pytest-cov` and `hypothesis` is required. Coverage of `top_words.py` by the combined suite must be **100 % line coverage**. Branch coverage is measured but is **not** a gate: the reference implementation contains a few defensive or rare-path branches (the "and N more" message in `_unsupported_error`, the supplementary-plane variation-selector range in `_is_variation_selector`) that the suite intentionally does not exercise, and these do not affect observable behavior for any accepted input.

---

## 3. Pipeline contract under test

Every end-to-end test goes through this exact call shape, which is the only supported public entry point:

```python

def top_words(text: str, *, max_length: int = 1_000_000) -> str:
    return format_frequency_list(
        top_k_frequent(
            normalize(validate_article_text(text, max_length=max_length)),
            10,
        )
    )
```

- `k = 10` is fixed; `top_words` must not expose `k`.
    
- `max_length` is forwarded to `validate_article_text`; `top_words` must not re-check it.
    
- Exceptions from `validate_article_text` propagate unchanged.
    
- `normalize` is called **after** validation, on the validator's return value, never on the raw input.
    

Any test that imports a component other than `top_words` is a unit test in §5–§8; any test that only imports `top_words` is an acceptance test in §9.

---

## 4. Shared fixtures (`conftest.py`)

```python

# tests/conftest.py
import pytest
from top_words import (
    ALLOWED_CHARS,
    InvalidArticleText,
    TextTooLongError,
    MalformedUnicodeError,
    UnsupportedScriptError,
    EmptyArticleTextError,
)
ALLOWED = frozenset(ALLOWED_CHARS)
@pytest.fixture
def boundary_long_text() -> str:
    """Exactly max_length characters; must be accepted."""
    return "a" * 1_000_000
@pytest.fixture
def just_over_boundary() -> str:
    return "a" * 1_000_001
```

`ALLOWED_CHARS` must be importable from `top_words.py` as a module-level constant (it is already in the error-handling section §9's reference code). Reuse it; do not redefine the alphabet in tests.

---

## 5. Normalization unit tests (`test_normalize.py`)

### 5.1 Documented vectors (normalization §6)

Every row in the normalization section's §6 table is a test case:

```python

import pytest
from top_words import normalize
@pytest.mark.parametrize("text,expected", [
    ("", ""),
    (" \t\n ", ""),
    ("!!!...???", ""),
    ("Hello, world!", "hello world"),
    ("Hello---world", "hello world"),
    ("don't stop", "don t stop"),
    ("COVID-19 cases", "covid cases"),
    ("a1b2c3", "a b c"),
    ("Привет, мир!", "привет мир"),
    ("Ёлка, ёж и ёрш", "ёлка ёж и ёрш"),
    ("Қазақстан Республикасы", "қазақстан республикасы"),
    ("Бағасы 1000 теңге!", "бағасы теңге"),
    ("word🔥word", "word word"),
    (" mixed\t\n script ", "mixed script"),
    ("naïve café", "na ve caf"),
    ("__init__", "init"),
    ("Мәтін – English & қазақ", "мәтін english қазақ"),
])
def test_normalize_documented_vectors(text, expected):
    assert normalize(text) == expected
```

### 5.2 Additional cases exercising the alphabet boundary

```python

@pytest.mark.parametrize("text,expected", [
    ("ӘӘӘ әәә", "әәә әәә"),               # both cases of a Kazakh-only letter
    ("Һ һ Һ һ", "һ һ һ һ"),
    ("ё Ё", "ё ё"),
    ("яЯ", "яя"),
    ("aA", "aa"),
    ("a\u00a0b", "a b"),                    # NBSP is a separator
    ("a\u2009b", "a b"),                    # thin space is a separator
    ("a\u0301b", "b"),                      # NFC composes to á (foreign letter) → separator
    ("💯", ""),
    ("Ⅳ", ""),                              # Roman numeral is not a letter
    ("ǅ", ""),                              # titlecase Latin letter not in §3 alphabet
    ("A1B", "a b"),                         # digits split, not glued
])
def test_normalize_alphabet_boundary(text, expected):
    assert normalize(text) == expected
```

### 5.3 Normalization invariants (normalization §5)

Directly assert invariants 1–6 of the normalization section on every vector above:

```python

from hypothesis import given, strategies as st
def _normalized_or_none(s):
    try:
        return normalize(s)
    except Exception:
        return None
@given(st.text())
def test_normalize_idempotent(s):
    n = _normalized_or_none(s)
    if n is None:
        return
    assert normalize(n) == n
@given(st.text())
def test_normalize_well_formed(s):
    n = _normalized_or_none(s)
    if n is None:
        return
    assert n == n.strip()
    assert "  " not in n
    if n:
        for tok in n.split(" "):
            assert tok != ""
            assert set(tok) <= ALLOWED
@given(st.text())
def test_normalize_lowercase_fixed_point(s):
    n = _normalized_or_none(s)
    if n is None:
        return
    assert n.lower() == n
@given(st.text())
def test_normalize_length_monotone(s):
    n = _normalized_or_none(s)
    if n is None:
        return
    assert len(n) <= len(s)
```

---

## 6. Validation unit tests (`test_validate.py`)

### 6.1 Documented vectors (error-handling §8)

```python

import pytest
from top_words import (
    validate_article_text,
    TextTooLongError, MalformedUnicodeError,
    UnsupportedScriptError, EmptyArticleTextError,
)
@pytest.mark.parametrize("text,expected_return", [
    ("Hello, world!", "Hello, world!"),
    ("Привет, мир!", "Привет, мир!"),
    ("Қазақстан Республикасы", "Қазақстан Республикасы"),
    ("Мәтін – English & қазақ", "Мәтін – English & қазақ"),
    ("1000 теңге", "1000 теңге"),
    ("и\u0306", "й"),                                    # NFC composes
])
def test_validate_accepts(text, expected_return):
    assert validate_article_text(text) == expected_return
@pytest.mark.parametrize("text,exc", [
    ("",                     EmptyArticleTextError),
    (" \t\n ",               EmptyArticleTextError),
    ("1234 !!! 😀",           EmptyArticleTextError),
    ("Ⅳ",                    EmptyArticleTextError),
    ("\u0301",               EmptyArticleTextError),
    ("😀",                    EmptyArticleTextError),
    ("❤\ufe0f",              EmptyArticleTextError),
    ("naïve",                UnsupportedScriptError),
    ("café",                 UnsupportedScriptError),
    ("cafe\u0301",           UnsupportedScriptError),
    ("Straße",               UnsupportedScriptError),
    ("Größe",                UnsupportedScriptError),
    ("Ελλάδα",               UnsupportedScriptError),
    ("日本語",                UnsupportedScriptError),
    ("Привет, naïve",        UnsupportedScriptError),
    ("q\u0307",              UnsupportedScriptError),
    ("x\x00y",               MalformedUnicodeError),
    ("\ud800",               MalformedUnicodeError),
    ("q\u0307\u0308",           UnsupportedScriptError),   # mark on mark: exercises `base -= 1`
    ("é\u0308",                UnsupportedScriptError),   # mark on a foreign letter: benign
])
def test_validate_rejects(text, exc):
    with pytest.raises(exc):
        validate_article_text(text)
def test_validate_length_just_over(boundary_long_text, just_over_boundary):
    # limit is inclusive
    assert validate_article_text(boundary_long_text) == boundary_long_text
    with pytest.raises(TextTooLongError):
        validate_article_text(just_over_boundary)
@pytest.mark.parametrize("bad", [None, b"abc", 123, ["a"], 1.5, True, object()])
def test_validate_type_errors(bad):
    with pytest.raises(TypeError):
        validate_article_text(bad)
```

### 6.2 Error attributes

Every documented attribute must be populated correctly:

```python

def test_text_too_long_attributes():
    try:
        validate_article_text("a" * 1_000_001)
    except TextTooLongError as e:
        assert e.length == 1_000_001
        assert e.max_length == 1_000_000
def test_malformed_nul_attributes():
    try:
        validate_article_text("x\x00y")
    except MalformedUnicodeError as e:
        assert e.kind == "nul"
        assert e.index == 1
def test_malformed_surrogate_attributes():
    try:
        validate_article_text("\ud800")
    except MalformedUnicodeError as e:
        assert e.kind == "surrogate"
        assert e.index == 0
def test_unsupported_script_offenders():
    try:
        validate_article_text("naïve café")
    except UnsupportedScriptError as e:
        # 'ï' and 'é' both reported; order by first occurrence
        chars = [c for _, c, _ in e.offenders]
        assert "ï" in chars and "é" in chars
        # messages must not echo article content
        for _, c, _ in e.offenders:
            assert c in "ïé"
```

### 6.3 Precedence contract (error-handling §4)

The check order is part of the contract and must be tested on inputs that fail several checks at once:

```python

def test_precedence_nul_over_script():
    with pytest.raises(MalformedUnicodeError) as ei:
        validate_article_text("naïve\x00")
    assert ei.value.kind == "nul"
def test_precedence_length_over_content():
    with pytest.raises(TextTooLongError):
        validate_article_text("naïve" + "a" * 2_000_000)
def test_precedence_script_over_empty():
    with pytest.raises(UnsupportedScriptError):
        validate_article_text("日本語")          # no supported letters, but script wins
def test_precedence_type_over_config():
    with pytest.raises(TypeError):
        validate_article_text(None, max_length=-1)
def test_config_error_not_article_error():
    with pytest.raises(ValueError) as ei:
        validate_article_text("hello", max_length=-1)
    assert not isinstance(ei.value, InvalidArticleText)
@pytest.mark.parametrize("bad_max", [-1, -1_000, True, False, 1.5, "100"])
def test_config_sanity(bad_max):
    with pytest.raises(ValueError) as ei:
        validate_article_text("hello", max_length=bad_max)
    assert not isinstance(ei.value, InvalidArticleText)
```

### 6.4 Validation invariants (error-handling §7)

```python

@given(st.text())
def test_validate_is_total(s):
    """Always returns a str or raises from the documented taxonomy."""
    try:
        x = validate_article_text(s)
    except InvalidArticleText:
        return
    except TypeError:
        return
    assert isinstance(x, str)
@given(st.text())
def test_validate_idempotent(s):
    try:
        x = validate_article_text(s)
    except (InvalidArticleText, TypeError):
        return
    assert validate_article_text(x) == x
@given(st.text())
def test_validate_output_is_nfc(s):
    try:
        x = validate_article_text(s)
    except (InvalidArticleText, TypeError):
        return
    import unicodedata
    assert unicodedata.is_normalized("NFC", x)
@given(st.text())
def test_validate_no_nul_no_surrogate(s):
    try:
        x = validate_article_text(s)
    except (InvalidArticleText, TypeError):
        return
    assert "\x00" not in x
    for ch in x:
        assert not (0xD800 <= ord(ch) <= 0xDFFF)
```

---

## 7. Word-counting unit tests (`test_top_k.py`)

### 7.1 Documented worked examples (word-counting §6)

```python

import pytest
from top_words import top_k_frequent
@pytest.mark.parametrize("text,k,expected", [
    ("i love leetcode i love coding", 2, [("i", 2), ("love", 2)]),
    ("the day is sunny the the the sunny is is", 2,
     [("the", 4), ("is", 3)]),
    ("", 10, []),
    ("   ", 10, []),
    ("a b c d", 2, [("a", 1), ("b", 1)]),               # lex tie-break
    ("a a a", 1, [("a", 3)]),
    ("a b c d", 0, []),                                  # k <= 0
    ("a b c d", -5, []),
])
def test_top_k_documented(text, k, expected):
    assert top_k_frequent(text, k) == expected
```

### 7.2 Tie-break at the k boundary

The most failure-prone case: when several candidates with equal count straddle position `k`, only the lexicographically smallest survive.

```python

def test_tie_break_at_boundary_lex_smallest_survives():
    # 5 words with count 3, 10 words with count 1, k=10
    text = "a a a b b b c c c d d d e e e f g h i j k l m n o"
    expected = [("a", 3), ("b", 3), ("c", 3), ("d", 3), ("e", 3),
                ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]
    assert top_k_frequent(text, 10) == expected
def test_tie_break_at_boundary_reverse_input_order():
    # Same multiset, inserted in the opposite order.
    text = "o n m l k j i h g f e e e d d d c c c b b b a a a"
    expected = [("a", 3), ("b", 3), ("c", 3), ("d", 3), ("e", 3),
                ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]
    assert top_k_frequent(text, 10) == expected
def test_all_tied_takes_lex_smallest():
    text = "l k j i h g f e d c b a"
    expected = [(w, 1) for w in "abcdefghij"]
    assert top_k_frequent(text, 10) == expected
```

### 7.3 k >= U returns everything, sorted

```python

def test_k_exceeds_unique():
    text = "x y z z"
    # counts: z=2, x=1, y=1; ordering: z first, then x,y lex
    assert top_k_frequent(text, 100) == [("z", 2), ("x", 1), ("y", 1)]
```

### 7.4 Determinism

Same input, same output, on repeat:

```python

def test_top_k_deterministic():
    s = "b a b a c c c d"
    assert top_k_frequent(s, 2) == top_k_frequent(s, 2)
    assert top_k_frequent(s, 2) == [("c", 3), ("a", 2)]
```

---

## 8. Formatting unit tests (`test_format.py`)

Every row from the formatting section's edge-case table:

```python

import pytest
from top_words import format_frequency_list
@pytest.mark.parametrize("entries,expected", [
    ([], ""),
    ([("cat", 3)], "cat: 3"),
    ([("a", 1), ("b", 1)], "a: 1, b: 1"),
    ([("a", 1), ("a", 2)], "a: 1, a: 2"),
    ([("x", 0), ("y", -1)], "x: 0, y: -1"),
    ([("cat", 3), ("dog", 2), ("bird", 1)],
     "cat: 3, dog: 2, bird: 1"),
])
def test_format_documented(entries, expected):
    assert format_frequency_list(entries) == expected
def test_format_preserves_order():
    entries = [("z", 1), ("a", 2), ("m", 3)]
    assert format_frequency_list(entries) == "z: 1, a: 2, m: 3"
```

---

## 9. End-to-end acceptance tests (`test_top_words.py`)

**This is the acceptance gate.** Every row is a contract between the whole pipeline and the outside world. If a row here fails, the implementation is not done, regardless of whether every unit test passes.

```python

import pytest
from top_words import (
    top_words,
    TextTooLongError, MalformedUnicodeError,
    UnsupportedScriptError, EmptyArticleTextError,
)
# ---- rejection propagates from validate ----
@pytest.mark.parametrize("text,exc", [
    ("",                              EmptyArticleTextError),
    ("   \t\n",                       EmptyArticleTextError),
    ("1234 !!! 😀",                   EmptyArticleTextError),
    ("naïve",                         UnsupportedScriptError),
    ("Привет, naïve",                 UnsupportedScriptError),
    ("日本語",                        UnsupportedScriptError),
    ("\ud800",                        MalformedUnicodeError),
    ("a" * 1_000_001,                 TextTooLongError),
])
def test_top_words_rejects(text, exc):
    with pytest.raises(exc):
        top_words(text)
# ---- exact output strings ----
@pytest.mark.parametrize("text,expected", [
    # single word
    ("cat", "cat: 1"),
    # punctuation and case collapsed
    ("Hello, world! Hello.", "hello: 2, world: 1"),
    ("The THE the", "the: 3"),
    # digits stripped, not tokenized into numbers
    ("cat 42 dog 42", "cat: 1, dog: 1"),
    # simple tie-break
    ("The cat. The cat.", "cat: 2, the: 2"),
    # full 10-slot output
    ("a b c d e f g h i j k",
     "a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1, j: 1"),
    # mixed English/Russian/Kazakh, all count 1; sorted by code point
    ("Мәтін – English & қазақ", "english: 1, мәтін: 1, қазақ: 1"),
    # Cyrillic-only
    ("Привет мир привет", "привет: 2, мир: 1"),
    # Kazakh-only
    ("Қазақ тілі қазақ", "қазақ: 2, тілі: 1"),
    # k-boundary tie-break
    ("a a a b b b c c c d d d e e e f g h i j k l m n o",
     "a: 3, b: 3, c: 3, d: 3, e: 3, f: 1, g: 1, h: 1, i: 1, j: 1"),
    # NFD input composed by the validator
    ("и\u0306 и\u0306", "й: 2"),
    # limit is inclusive
    ("a" * 500_000, "a: 500000"),
])
def test_top_words_exact_output(text, expected):
    assert top_words(text) == expected
```

The rows above must reproduce **byte-for-byte**. Any deviation — a wrong count, a wrong tie-break, a missing trailing separator behavior, a Cyrillic letter shown in the wrong case, an ASCII/Cyrillic sort boundary misidentified — is a failure.

### 9.1 Cross-checks between top_words and its components

```python

def test_top_words_agrees_with_manual_pipeline():
    samples = [
        "The cat sat on the mat.",
        "Привет мир привет",
        "Қазақ тілі қазақ",
        "a b c a b c a b c",
        "word🔥word word",
    ]
    from top_words import (
        validate_article_text, normalize,
        top_k_frequent, format_frequency_list,
    )
    for s in samples:
        assert top_words(s) == format_frequency_list(
            top_k_frequent(normalize(validate_article_text(s)), 10)
        )
```

This pins down that `top_words` is _exactly_ the composition in §3 — not a re-implementation that happens to agree.

---

## 10. Invariant / property tests (`test_invariants.py`)

### 10.1 Pipeline-level invariants

```python

from hypothesis import given, strategies as st, settings
import unicodedata
from top_words import top_words, InvalidArticleText, normalize, validate_article_text
@given(st.text())
def test_top_words_returns_str_or_documented_error(s):
    try:
        out = top_words(s)
    except InvalidArticleText:
        return
    except TypeError:
        return
    assert isinstance(out, str)
    # Structural check: "word: N" entries separated by ", "
    if out:
        for entry in out.split(", "):
            word, _, count = entry.rpartition(": ")
            assert word, f"empty word in {entry!r}"
            assert word == word.lower()
            assert int(count) > 0
@given(st.text())
def test_top_words_deterministic(s):
    try:
        a = top_words(s)
    except InvalidArticleText:
        return
    except TypeError:
        return
    b = top_words(s)
    assert a == b
@given(st.text())
def test_top_words_never_more_than_ten(s):
    try:
        out = top_words(s)
    except (InvalidArticleText, TypeError):
        return
    assert out.count(", ") <= 9  # at most 10 entries
```

### 10.2 Gate property (error-handling §7.2)

If validation succeeds, `normalize` must not raise on the validator's output:

```python

@given(st.text())
def test_normalize_never_raises_after_validation(s):
    try:
        x = validate_article_text(s)
    except (InvalidArticleText, TypeError):
        return
    # must not raise
    normalize(x)
```

### 10.3 Agreement property (error-handling §7.3)

```python

@given(st.text())
@settings(max_examples=500)
def test_validate_normalize_agreement(s):
    try:
        x = validate_article_text(s)
    except (InvalidArticleText, TypeError):
        return
    assert normalize(x) == normalize(s)
```

This invariant holds because `normalize` performs NFC normalization before tokenizing (normalization §4 rule 0), so `normalize(x) == normalize(s)` for any `s` whose validated form is `x`.

---

## 11. Determinism and idempotence checks (cross-cutting)

```python

def test_repeated_runs_byte_identical():
    cases = [
        "The cat sat on the mat.",
        "Привет мир привет",
        "Қазақ тілі қазақ",
        "a b c a b c a b c",
        "word🔥word word",
        "Мәтін – English & қазақ",
    ]
    for s in cases:
        assert top_words(s) == top_words(s) == top_words(s)
def test_repeated_same_input_deterministic():
    # Same input, same output, on repeat. (top_words' output is NOT fed back in as
    # article input: it contains colons and digits, which normalize would strip.)
    s = "The cat sat on the mat. The cat!"
    out1 = top_words(s)
    out2 = top_words(s)
    assert out1 == out2
```

---

## 12. Resolved design decisions

Two ambiguities surfaced by this suite are resolved in the spec (not in the tests):

### 12.1 `normalize` performs NFC normalization

`normalize` now applies `unicodedata.normalize("NFC", text)` before the separator substitution (normalization §4 rule 0 and §7). This makes invariant §7.3 (`normalize(validate_article_text(t)) == normalize(t)`) hold, because the validator and the normalizer agree on the canonical form. A consequence: `normalize("e\u0301") == ""`, because NFC composes to `é`, which is outside §3 and therefore a separator. This is acceptable because `validate_article_text` rejects `"e\u0301"` before `normalize` sees it.

### 12.2 `normalize` strips foreign letters rather than raising

`normalize` is total and never raises: letters outside §3 are separators, not errors (normalization §6 vector 15). The script check lives exclusively in `validate_article_text`. This is the "strip" resolution recommended in review; it keeps `normalize` a pure transform and matches the assignment's goal of one plausible implementation.

---

## 13. Acceptance gate

An implementation is accepted iff:

1. `pytest tests/` passes with `--strict-markers` (no `xfail`, no `skip` except the slow test in §14).
    
2. `pytest --cov=top_words --cov-fail-under=100` reports 100 % line coverage.
    
3. `test_top_words_exact_output` (§9) passes byte-for-byte on all 12 rows.
    
4. `test_validate_normalize_agreement` (§10.3) passes.
    
5. `test_top_words_agrees_with_manual_pipeline` (§9.1) passes.
    
6. The output of `top_words` matches the spec's stated output format `"word: N, word: N, …"` with no trailing separator and no leading space.
    

Any implementation that passes the unit tests but fails any row of §9 is not accepted: the unit tests prove each component matches its own section, but only §9 proves the components wire together the way the assignment demands.

---

## 14. Optional but recommended: performance smoke test

Marked slow and excluded from the default run:

```python

import time
import pytest
@pytest.mark.slow
def test_large_input_under_limit():
    s = "word " * 199_000  # 995,000 chars, under the 1M default limit
    t0 = time.perf_counter()
    out = top_words(s)
    dt = time.perf_counter() - t0
    assert out == "word: 199000"
    assert dt < 5.0  # generous bound; guards against O(n²) regressions
```

Implementation

Input normalization

## 1. Purpose

Define a single, deterministic preprocessing step that converts _any_ Python `str` (article text) into a canonical whitespace-separated sequence of word tokens containing **only letters** from the English, Russian, or Kazakh alphabets. All downstream logic (frequency counting, top-10 selection) operates exclusively on this normalized form, so it never has to deal with case, punctuation, digits, or scripts.

## 2. Interface

text

normalize(text: str) -> str

- **Input:** any `str`, including empty, whitespace-only, mixed-script, emoji, digits, control characters, very long text.
    
- **Output:** a `str` matching `^$|^[letters]+( [letters]+)*$`, i.e. tokens of letters separated by exactly one ASCII space (U+0020), no leading/trailing space.
    
- **Purity:** no I/O, no global state, no locale dependence, input not mutated. Same input ⇒ same output, on every platform and Python version.
    

The output string is also the token stream: `tokens = normalized.split()` (never `split(" ")` on the raw text).

## 3. Letter set (the only characters that survive)

|Script|Characters kept (lowercase form)|
|---|---|
|English|`a`–`z`|
|Russian|`а`–`я`, `ё`|
|Kazakh-specific|`ә ғ қ ң ө ұ ү һ і`|

Uppercase counterparts of all of the above are also accepted as input letters. Nothing else is a letter — not `é`, not `ü`, not `ñ`, not CJK, not Greek.

Regex class (uppercase + lowercase explicitly enumerated; `ё` and Kazakh letters fall outside the `А-Я`/`а-я` ranges and must be listed):

```
python

LETTERS = "A-Za-zА-Яа-яЁёӘәҒғҚқҢңӨөҰұҮүҺһІі"
NON_LETTER_RUN = re.compile(rf"[^{LETTERS}]+")
```

## 4. Rules (applied strictly in this order)

0. **NFC normalization first.** Apply `unicodedata.normalize("NFC", text)` before anything else. This composes decomposed sequences (e.g. `и\u0306` → `й`) so that letter tokens are canonical before separator substitution. A documented consequence: `normalize("e\u0301") == ""` — NFC composes to `é`, which is outside §3 and therefore a separator. This is acceptable because `validate_article_text` rejects `"e\u0301"` before `normalize` sees it.
    

1. **Non-letters become separators.** Every maximal run of characters that are not in the letter set (§3) is replaced by a **single** space. This covers punctuation, digits, hyphens, underscores, apostrophes, quotes, brackets, math/currency symbols, emoji, and all whitespace (`\t`, `\n`, `\r`, NBSP U+00A0, thin space U+2009, etc.).
    
    - Consequence: a separator run never produces two adjacent spaces and never produces an empty token.
        
    - Consequence: text with no letters at all collapses to the empty string.
        
2. **Trim.** Remove leading and trailing spaces.
    
3. **Lowercase last.** Apply `str.lower()` to the trimmed result. Lowercasing happens _after_ filtering so that exotic case mappings (e.g. `İ`, `ß`) can never manufacture letters or change token boundaries — such characters were already destroyed by rule 1.
    

Rules 0–3 are equivalent to the reference implementation in §7.

### Deliberate decisions (documented, not accidental)

|Input|Output|Rationale|
|---|---|---|
|`don't`|`don t`|Apostrophe is a separator, not a joiner. "Keep letters, spaces in between" is applied literally.|
|`well-known`|`well known`|Hyphen is a separator; compound splits into two words.|
|`COVID-19`|`covid`|`19` is a separator run; digits are dropped, not glued.|
|`a1b2c3`|`a b c`|Digits split letter runs.|
|`Ёлка`|`ёлка`|`ё` is a letter; case folded.|
|`Қазақстан`|`қазақстан`|Kazakh letters preserved and folded.|

If a product decision later wants `don't → dont`, that is a **change to rule 1** (apostrophe deleted rather than replaced by space) and must be versioned, not patched locally in the counter.

## 5. Invariants (property-test these)

1. **Idempotent:** `normalize(normalize(s)) == normalize(s)`.
    
2. **Well-formed:** output matches `^$|^[letters]+( [letters]+)*$`.
    
3. **No empty tokens:** `" "` never appears in the output; `normalized.split()` has no `""` element.
    
4. **Token-preserving under separator variation:** inserting/removing punctuation, digits, or arbitrary whitespace between two letter runs does not change the token list.
    
5. **Case-insensitive:** `normalize(s.upper()) == normalize(s)` for inputs whose letters come from §3.
    
6. **Monotone length:** `len(normalize(s)) <= len(s)` (lowercasing of in-set letters never lengthens).
    
7. **Complexity:** single pass, O(n) time and memory in the input length; no per-character Python loop required.
    

## 6. Test vectors

|#|Input|Expected output|
|---|---|---|
|1|`""`|`""`|
|2|`" \t\n "`|`""`|
|3|`"!!!...???"`|`""`|
|4|`"Hello, world!"`|`"hello world"`|
|5|`"Hello---world"`|`"hello world"`|
|6|`"don't stop"`|`"don t stop"`|
|7|`"COVID-19 cases"`|`"covid cases"`|
|8|`"a1b2c3"`|`"a b c"`|
|9|`"Привет, мир!"`|`"привет мир"`|
|10|`"Ёлка, ёж и ёрш"`|`"ёлка ёж и ёрш"`|
|11|`"Қазақстан Республикасы"`|`"қазақстан республикасы"`|
|12|`"Бағасы 1000 теңге!"`|`"бағасы теңге"`|
|13|`"word🔥word"`|`"word word"`|
|14|`" mixed\t\n script "`|`"mixed script"`|
|15|`"naïve café"`|`"na ve caf"`|
|16|`"__init__"`|`"init"`|
|17|`"Мәтін – English & қазақ"`|`"мәтін english қазақ"`|

(Vector 15 shows the documented consequence that letters outside §3 are separators; if Latin-1 accented letters must be folded to base letters, that is a separate, explicitly versioned rule added _before_ rule 1.)

## 7. Reference implementation
```
python

import re
import unicodedata
LETTERS = "A-Za-zА-Яа-яЁёӘәҒғҚқҢңӨөҰұҮүҺһІі"
_NON_LETTER_RUN = re.compile(rf"[^{LETTERS}]+")
def normalize(text: str) -> str:
    """Canonicalize article text to lowercase letter-tokens separated by single spaces."""
    return _NON_LETTER_RUN.sub(" ", unicodedata.normalize("NFC", text)).strip().lower()

`unicodedata.normalize("NFC", …)` is rule 0; `re.sub` with a `+`-quantified class performs rule 1 and the collapse-to-one-space in a single pass; `strip()` is rule 2; `lower()` is rule 3.```

Word counting

## 1. Problem Statement

Given a normalized string `str` (words separated by whitespace, no punctuation, no special characters) and an integer `k`, return the `k` most frequent words ordered by **frequency descending**, with ties broken by **lexicographically ascending**.

Return the result as a `List[Tuple[word: str, count: int]]`.

## 2. Input / Output Contract

||Description|
|---|---|
|**Input**|`str: str` — normalized text; only alphabetic words and whitespace|
|**Input**|`k: int` — desired number of top words|
|**Output**|`List[Tuple[str, int]]` — length `min(k, U)`, where `U` = number of unique words|
|**Ordering**|Primary: `count` descending · Secondary: `word` lexicographically ascending|
|**Pre-conditions**|`str` contains no punctuation, digits, or special characters|
|**Post-conditions**|Every tuple `(w, c)` has `c = frequency of w in str`; no word appears twice; length is `min(k, U)`|

## 3. Approach: Hash Map + Min-Heap

**Idea.** Build a frequency map of all words. Then maintain a **min-heap of size `k`** whose root is the _"worst"_ candidate currently in the running top-k. When a new word arrives and the heap is full, compare it against the root: if it is better, pop the root and push the new word; otherwise discard. At the end, the heap contains exactly the `k` best candidates (in arbitrary order).

**Why a min-heap?** We cannot afford a full sort of all `U` unique words (`O(U log U)`). A size-`k` min-heap keeps only the survivors and costs `O(U log k)` — the key optimization for the stated `O(N log K)` bound.

## 4. Algorithm
```

text

function topK(str, k) -> List[(word, count)]:
    # Step 1 — Tokenize
    tokens ← str.split()            # split on whitespace
    # Step 2 — Count frequencies (hash map)
    freq ← empty HashMap
    for each token t in tokens:
        freq[t] ← freq.get(t, 0) + 1
    # Step 3 — Min-heap of size k
    heap ← empty MinHeap             # entries: (count, ReverseLexWord)
    for each (w, c) in freq:
        heap.push( (c, ReverseLexWord(w)) )
        if heap.size() > k:
            heap.pop()               # evict "worst" element (see §5)
    # Step 4 — Extract and sort into required output order
    result ← [ (rw.word, c) for (c, rw) in heap ]
    result.sort( key = (count desc, word asc) )
    return result
```

## 5. Heap Comparator (Critical Detail)

A plain `(count, word)` tuple will **not** work, because Python/Java/Python heaps are _min_-heaps: the root pops the smallest `count`, and on ties the _lexicographically smallest_ word — but our tie-break rule says the lexicographically **largest** word is the worst candidate and should be evicted first.

**Define `ReverseLexWord(w)`** as a wrapper whose comparison is inverted:

text

ReverseLexWord(a) < ReverseLexWord(b)   ⇔   a > b   (lexicographically)

With tuples `(count, ReverseLexWord(word))` and the native min-heap ordering, the root satisfies:

> **Root = word with smallest count; on equal count, the lexicographically largest word.**

This is exactly the element that is _furthest_ from belonging to the top-k (since the top-k prefers high count, then lexicographically _small_). Popping it when `size > k` preserves the correct top-k.

> **Language note.**
> 
> - **Python:** implement `ReverseLexWord` as a class with `__lt__` returning `self.s > other.s`.
>     

## 6. Worked Example

**Input:** `str = "i love leetcode i love coding"`, `k = 2`

|Word|Count|
|---|---|
|i|2|
|love|2|
|leetcode|1|
|coding|1|

Heap trace (entries shown as `(count, ReverseLexWord)`):

|Push|Heap (root = min)|Evict|
|---|---|---|
|(2, i)|[(2, i)]|—|
|(2, love)|[(2, i), (2, love)]|—|
|(1, leetcode)|3 items → pop `(1, leetcode)`|leetcode|
|(1, coding)|3 items → pop `(1, coding)`|coding|

Final heap = `[(2, i), (2, love)]`. Sort by (count desc, word asc) → **`[(i, 2), (love, 2)]`**. ✅ (`i < love`, so `i` first on the tie.)

**Input:** `str = "the day is sunny the the the sunny is is"`, `k = 2`  
Counts: `the=4, is=3, sunny=2, day=1` → **`[(the, 4), (is, 3)]`**.

## 7. Edge Cases

|Case|Behavior|
|---|---|
|`str == ""` or whitespace-only|`tokens = []`, `U = 0` → return `[]`|
|`k <= 0`|Return `[]`|
|`k >= U`|Return all unique words, fully sorted by (count desc, word asc)|
|All counts tied (`"a b c d"`, `k=2`)|Tie-break lexicographic: `[(a,1), (b,1)]`|
|Duplicate words only (`"a a a"`, `k=1`)|`[(a, 3)]`|

## 8. Complexity

Let `N` = number of tokens, `U` = number of unique words (`U ≤ N`), `k` = requested size.

|Phase|Time|Space|
|---|---|---|
|Tokenize + count|`O(N)`|`O(U)`|
|Heap build (U pushes, each `O(log k)`; up to U pops)|`O(U log k)`|`O(k)`|
|Final sort of heap (`O(k log k)`)|`O(k log k)`|`O(k)`|
|**Total**|**`O(N + U log k)` = `O(N log k)`**|**`O(U + k)` = `O(U)`**|

Because `U ≤ N`, the stated bound `O(N log K)` holds.

## 9. Reference Implementation (Python)
```

python

from collections import defaultdict
import heapq
class ReverseLexWord:
    __slots__ = ("s",)
    def __init__(self, s: str):
        self.s = s
    def __lt__(self, other: "ReverseLexWord") -> bool:
        return self.s > other.s            # inverted ordering
def top_k_frequent(str_input: str, k: int) -> list[tuple[str, int]]:
    if k <= 0:
        return []
    # Step 1–2: tokenize and count
    freq: dict[str, int] = defaultdict(int)
    for w in str_input.split():
        freq[w] += 1
    # Step 3: min-heap of size k
    heap: list[tuple[int, ReverseLexWord]] = []
    for w, c in freq.items():
        heapq.heappush(heap, (c, ReverseLexWord(w)))
        if len(heap) > k:
            heapq.heappop(heap)
    # Step 4: extract and sort into output order
    result = [(rw.s, c) for c, rw in heap]
    result.sort(key=lambda t: (-t[1], t[0]))
    return result
```
```
**Example check:**

```

```
python

top_k_frequent("i love leetcode i love coding", 2)
# → [('i', 2), ('love', 2)]
top_k_frequent("the day is sunny the the the sunny is is", 2)
# → [('the', 4), ('is', 3)]
```

Return

### 1. Purpose

Transform a `List[Tuple[str, int]]` of `(word, count)` entries into a human-readable display string of the form:

text

"cat: 3, dog: 2, bird: 1"

This is a pure formatting operation. It does **not** sort, aggregate, deduplicate, filter, or validate frequencies.

---

### 2. Input / Output Contract

|Item|Description|
|---|---|
|Input|`entries: List[Tuple[str, int]]` — each tuple is `(word, count)`|
|Output|`str` — comma-and-space separated display of each entry|
|Ordering|Preserve the exact input order|
|Separator|`", "` between entries|
|Entry format|`"{word}: {count}"`|
|Empty input|Return `""`|
|Single entry|Return `"{word}: {count}"` with no separator|

---

### 3. Preconditions

- `entries` is a list of 2-tuples.
    
- Each tuple’s first element is a `str`.
    
- Each tuple’s second element is an `int`.
    
- No escaping or quoting is required for words.
    
- Counts are formatted using standard decimal representation.
    

---

### 4. Postconditions

For input `entries` of length `n`:

- If `n == 0`, output is `""`.
    
- Otherwise, output contains exactly `n` formatted entries.
    
- Entry `i` in the output is exactly:
    

text

entries[i][0] + ": " + str(entries[i][1])

- Entries are joined by `", "`.
    
- No leading or trailing separator is added.
    
- Duplicate words are preserved as separate entries if present in the input.
    
- Input order is preserved exactly.
    

---

### 5. Algorithm

text

function format_frequency_list(entries) -> str:
    if entries is empty:
        return ""
    parts ← empty list
    for each (word, count) in entries:
        parts.append(word + ": " + str(count))
    return join(parts, ", ")

---

### 6. Worked Example

Input:

python

[("cat", 3), ("dog", 2), ("bird", 1)]

Transformation:

|Index|Tuple|Formatted Entry|
|---|---|---|
|0|`("cat", 3)`|`"cat: 3"`|
|1|`("dog", 2)`|`"dog: 2"`|
|2|`("bird", 1)`|`"bird: 1"`|

Joined with `", "`:

text

"cat: 3, dog: 2, bird: 1"

---

### 7. Edge Cases

|Input|Output|
|---|---|
|`[]`|`""`|
|`[("cat", 3)]`|`"cat: 3"`|
|`[("a", 1), ("b", 1)]`|`"a: 1, b: 1"`|
|`[("a", 1), ("a", 2)]`|`"a: 1, a: 2"`|
|`[("x", 0), ("y", -1)]`|`"x: 0, y: -1"`|

---

### 8. Complexity

Let `n = len(entries)`.

|Phase|Time|Space|
|---|---|---|
|Format each entry|`O(n)`|`O(n)` for output parts|
|Join parts|`O(n)`|`O(n)` for final string|
|Total|`O(n)`|`O(n)`|

---

### 9. Reference Implementation

python

def format_frequency_list(entries: list[tuple[str, int]]) -> str:
    return ", ".join(f"{word}: {count}" for word, count in entries)

Example:

python

format_frequency_list([("cat", 3), ("dog", 2), ("bird", 1)])
# → "cat: 3, dog: 2, bird: 1"

Error handling

Input error handling

## 1. Purpose and position in the pipeline

v2 already raises `TypeError`/`ValueError`, but it raises them **as a side effect of trying to normalize**. That is the wrong place to enforce product policy: the counter, the HTTP handler, and any batch job each end up deciding what "bad input" means, and a caller can bypass the check by calling helpers directly.

This spec moves the decision to **one explicit gate** that runs first:

text

raw user input
      │
      ▼
validate_article_text(text)   ← THIS SPEC: accept or raise, nothing else
      │  returns NFC-normalized text (or raises)
      ▼
normalize(text)               ← v2 spec: pure transform, never raises on validated input
      │  returns "word word word"
      ▼
count / top-10                ← separate spec

**Rule:** product code (HTTP handler, batch job, CLI) MUST call `validate_article_text` and MUST pass its **return value** — not the raw input — to `normalize`. Calling `normalize` directly on unvalidated input is a wiring bug, not a supported path (see §11).

## 2. Interface

python

validate_article_text(text: str, *, max_length: int = 1_000_000) -> str

- **Returns:** the **NFC-normalized** form of `text`. Callers must use the return value; the input object is not returned and not mutated.
    
- **Raises:** exactly one of the exceptions in §3, or nothing.
    
- **Purity:** no I/O, no logging, no globals, deterministic. Same input ⇒ same return value or same exception class with the same attributes.
    
- **Cost:** O(n) time and memory in `len(text)`.
    

This function is a **validator, not a sanitizer**. It does not make text safe for HTML, SQL, shell, or logs; escaping/encoding is a separate concern.

## 3. Error taxonomy

All content errors derive from `InvalidArticleText(ValueError)`, so `except ValueError` keeps working exactly as promised in v2, while `except InvalidArticleText` distinguishes _bad article_ from _bad config_.

|Exception|Base|Raised when|Attributes|Message|
|---|---|---|---|---|
|`TypeError`|—|`text` is not a `str`|—|`text must be str, not <type>`|
|`InvalidArticleText`|`ValueError`|_(abstract base — never raised directly)_|—|—|
|`TextTooLongError`|`InvalidArticleText`|`len(text) > max_length`|`.length`, `.max_length`|`article text is too long: N characters exceeds the limit of M`|
|`MalformedUnicodeError`|`InvalidArticleText`|NUL character, or a lone surrogate|`.kind` (`"nul"`/`"surrogate"`), `.index`|`article text contains a NUL character (U+0000) at index I` / `article text contains a lone surrogate (U+D800) at index I`|
|`UnsupportedScriptError`|`InvalidArticleText`|letter material outside English/Russian/Kazakh (v2 §2)|`.offenders` = `((index, char, category), …)`|`text contains characters outside the supported alphabets (English, Russian, Kazakh): …` (v2 §4 format)|
|`EmptyArticleTextError`|`InvalidArticleText`|zero supported letters after NFC|—|`article text contains no supported letters (English, Russian, Kazakh)`|

Notes:

- `UnsupportedScriptError` from v2 is **re-based** under `InvalidArticleText`. This is source-compatible: `except ValueError` still catches it.
    
- Messages **never echo article content**. Offenders are shown as `repr` of a single character plus its code point and category, capped as in v2 §4.
    
- `.index` and `.offenders` indices are positions in the **NFC form**, which may differ from raw-input indices for decomposed input (e.g. NFD `cafe\u0301`). Documented, not a bug.
    
- An invalid `max_length` argument (negative, `bool`, non-int) raises plain `ValueError`, **not** an `InvalidArticleText` — it is a programmer error, not an article error.
    

## 4. Check order (deterministic precedence)

Exactly one exception is raised per call. When several problems coexist, the earliest check wins:

|#|Check|Cost|Raises|
|---|---|---|---|
|1|`isinstance(text, str)`|O(1)|`TypeError`|
|2|`max_length` sanity|O(1)|`ValueError` (config)|
|3|`len(text) > max_length`|O(1)|`TextTooLongError`|
|4|`"\x00" in text`|O(n), C-level|`MalformedUnicodeError(kind="nul")`|
|5|lone-surrogate regex search|O(n), C-level|`MalformedUnicodeError(kind="surrogate")`|
|6|NFC normalize + script scan|O(n)|`UnsupportedScriptError`|
|7|at least one supported letter|O(n), C-level|`EmptyArticleTextError`|

Rationale for the ordering: cheap and unambiguous checks precede expensive ones; **length before content** so a 500 MB payload is rejected without scanning; **foreign script before empty** so `"日本語"` reports the real problem instead of "no letters". Length is measured on the raw input, not the NFC form (NFC may change length by a bounded amount via composition/exclusion).

## 5. Criteria in detail

### 5.1 `TypeError` — not a string

Reject **every** value for which `isinstance(value, str)` is false: `None`, `bytes`, `bytearray`, `int`, `float`, `bool`, `list`, `dict`, file objects, ORM field objects, `numpy.str_`, objects with a `__str__`. **No coercion.** `bytes` is rejected even if it decodes cleanly — decoding is the caller's explicit decision (and its failure mode must be visible there, not silently swallowed here).

`str` subclasses are accepted and treated as their value. Behavior of subclasses that override `lower()`/`__getitem__` is undefined and out of scope.

### 5.2 `TextTooLongError` — size cap

`len(text) > max_length`. The limit is **inclusive**: `len(text) == max_length` passes. Default 1 000 000 characters (≈ a large feature article; guards against accidental whole-corpus or repeated-paste input).

### 5.3 `MalformedUnicodeError` — text that is not well-formed Unicode

- **`kind="nul"`:** any U+0000. NUL in article text almost always signals truncation, C-string boundary, or an encoding bug; silently stripping it hides data loss, so it is rejected rather than normalized away.
    
- **`kind="surrogate"`:** any code unit in U+D800–U+DFFF (lone surrogates from `surrogateescape`/`surrogatepass` decoding). Such strings are not encodable as UTF-8 and cannot be persisted or logged reliably.
    

### 5.4 `UnsupportedScriptError` — foreign letter material

Identical criteria to v2 §2, applied to the NFC form: any `L*` character outside the supported set, or any non-variation-selector `M*` mark attached to a supported letter. `naïve`, `café`, `Straße`, `Größe`, `Ελλάδα`, `日本語`, `q̇` are all rejected. A single foreign letter rejects the whole string — no partial acceptance.

### 5.5 `EmptyArticleTextError` — nothing to count

After the foreign-script check passes, the NFC text contains no supported letter. This covers `""`, whitespace-only, punctuation-only, digit-only (`"1000"`), emoji-only (`"😀"`, `"❤️"`), `Nl` numerals (`"Ⅳ"`), and stray combining marks with no base (`"\u0301"`). Such input can only produce an empty trending list, which is a meaningless result; the boundary says so explicitly instead of returning `[]`.

## 6. Explicitly NOT errors

These are stripped by v2 and are **not** grounds for rejection. Listed so the boundary does not creep into a sanitizer:

|Category|Examples|Why allowed|
|---|---|---|
|Digits / `Nd`, `Nl`|`19`, `1000`, `Ⅳ`|numbers carry no trend signal; stripped|
|Punctuation|`.,;:!?—–-'"()[]{}«»…`|stripped|
|Symbols / emoji|`$ € © ™ 😀 🔥`|stripped|
|Whitespace|space, `\t`, `\n`, `\r`, NBSP U+00A0, thin space|stripped|
|Control chars (except NUL)|U+0001–U+001F, DEL|stripped|
|Bidi marks, ZWJ, ZWNJ|U+200B–U+200F, U+202A–U+202E, U+200D|stripped|
|Variation selectors|U+FE0F (in `❤️`, `1️⃣`)|presentation only, never part of a letter|
|Combining marks on non-letters|`1️⃣`'s keycap mark, marks on emoji|formatting junk, stripped|
|`Cf`/`Co`/`Cs`… (other than surrogates)|private-use chars|stripped|

Mixed-script _supported_ text is fine: `"Мәтін – English & қазақ"` passes.

## 7. Contract and invariants

1. **Totality:** for every Python object, `validate_article_text` returns a `str` or raises exactly one exception from §3.
    
2. **Gate property:** if `validate_article_text(t)` returns `x`, then `normalize(x)` does **not** raise. (This is the whole point — no wrong string reaches normalization.)
    
3. **Agreement:** `normalize(validate_article_text(t)) == normalize(t)` for every `t` on which validation succeeds. Validation changes policy, never words.
    
4. **Post-conditions on the return value `x`:** `x` is NFC; contains no NUL, no lone surrogate, no foreign letter material; contains at least one supported letter; `len` is within limits (up to NFC expansion).
    
5. **Idempotence:** `validate_article_text(validate_article_text(t)) == validate_article_text(t)`, and the second call never raises.
    
6. **Determinism:** identical input and `max_length` ⇒ identical outcome, identical message text, identical `.offenders`/`.index`.
    
7. **No mutation / no I/O / no logging.** Logging is the caller's job (§10).
    
8. **Stable precedence:** the check order in §4 is part of the contract; tests may assert which error wins on multi-fault input.
    

## 8. Test vectors

|#|Input|Outcome|
|---|---|---|
|1|`""`|`EmptyArticleTextError`|
|2|`" \t\n "`|`EmptyArticleTextError`|
|3|`"1234 !!! 😀"`|`EmptyArticleTextError`|
|4|`"Ⅳ"`|`EmptyArticleTextError`|
|5|`"\u0301"` (stray mark)|`EmptyArticleTextError`|
|6|`"😀"` / `"❤️"`|`EmptyArticleTextError`|
|7|`"Hello, world!"`|ok, returns as-is|
|8|`"Привет, мир!"`|ok|
|9|`"Қазақстан Республикасы"`|ok|
|10|`"Мәтін – English & қазақ"`|ok|
|11|`"1000 теңге"`|ok|
|12|`"naïve"`|`UnsupportedScriptError`, offender `ï`|
|13|`"café"`|`UnsupportedScriptError`|
|14|`"cafe\u0301"` (NFD)|`UnsupportedScriptError` — NFC composes to `é`|
|15|`"Straße"`, `"Größe"`|`UnsupportedScriptError`|
|16|`"Ελλάδα"`, `"日本語"`|`UnsupportedScriptError`|
|17|`"Привет, naïve"`|`UnsupportedScriptError` — supported text does not rescue the string|
|18|`"q\u0307"`|`UnsupportedScriptError` — attached, uncomposable mark|
|19|`"и\u0306"`|ok — NFC composes to supported `й`|
|20|`"x\x00y"`|`MalformedUnicodeError(kind="nul", index=1)`|
|21|`"\ud800"`|`MalformedUnicodeError(kind="surrogate", index=0)`|
|22|`"a" * 1_000_001`|`TextTooLongError(length=1000001)`|
|23|`"a" * 1_000_000`|ok (limit inclusive)|
|24|`"\x00" + "é"`|`MalformedUnicodeError` — NUL outranks script check|
|25|`None`, `b"abc"`, `123`, `["a"]`|`TypeError`|
|26|`max_length=-1`|`ValueError` (config; not `InvalidArticleText`)|

## 9. Reference implementation
```

python

import re
import unicodedata
# --- shared with the normalization spec: one source of truth for the alphabet ---
# A fixed string (not frozenset iteration) keeps messages deterministic across runs.
ALLOWED_CHARS = (
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    "АБВГДЕЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ"
    "абвгдежзийклмнопрстуфхцчшщъыьэюя"
    "ЁёӘәҒғҚқҢңӨөҰұҮүҺһІі"
)
_ALLOWED = frozenset(ALLOWED_CHARS)
_HAS_ALLOWED = re.compile("[" + ALLOWED_CHARS + "]")
_NON_ASCII = re.compile(r"[^\x00-\x7F]")       # ASCII: a–z A–Z allowed, rest non-letter
_SURROGATE = re.compile(r"[\ud800-\udfff]")
_VARIATION_SELECTORS = ((0xFE00, 0xFE0F), (0xE0100, 0xE01EF))
_MAX_REPORTED = 10
DEFAULT_MAX_LENGTH = 1_000_000
class InvalidArticleText(ValueError):
    """Base for all content errors. `except ValueError` still catches all of them."""
class TextTooLongError(InvalidArticleText):
    def __init__(self, message, *, length, max_length):
        super().__init__(message)
        self.length, self.max_length = length, max_length
class MalformedUnicodeError(InvalidArticleText):
    def __init__(self, message, *, kind, index):
        super().__init__(message)
        self.kind, self.index = kind, index
class UnsupportedScriptError(InvalidArticleText):
    def __init__(self, message, offenders):
        super().__init__(message)
        self.offenders = offenders
class EmptyArticleTextError(InvalidArticleText):
    pass
def _is_variation_selector(ch: str) -> bool:
    cp = ord(ch)
    return any(lo <= cp <= hi for lo, hi in _VARIATION_SELECTORS)
def _scan_script(nfc_text: str):
    """Return (offenders, first_seen) for foreign letter material. v2 §2 rules."""
    offenders, first_seen = [], {}
    for m in _NON_ASCII.finditer(nfc_text):
        i, ch = m.start(), m.group()
        if ch in _ALLOWED:
            continue
        cat = unicodedata.category(ch)
        if cat[0] == "M":
            if _is_variation_selector(ch):
                continue
            base = i - 1
            while base >= 0 and unicodedata.category(nfc_text[base])[0] == "M":
                base -= 1
            if not (base >= 0 and nfc_text[base] in _ALLOWED):
                continue                      # stray / emoji-attached mark: benign
        elif cat[0] != "L":
            continue                          # digits, punctuation, symbols, controls
        offenders.append((i, ch, cat))
        first_seen.setdefault(ch, i)
    return offenders, first_seen
def _unsupported_error(offenders, first_seen) -> UnsupportedScriptError:
    shown = list(first_seen.items())[:_MAX_REPORTED]
    details = "; ".join(
        f"{ch!r} (U+{ord(ch):04X}, {unicodedata.category(ch)}) at index {idx}"
        for ch, idx in shown
    )
    hidden = len(first_seen) - len(shown)
    more = f"; and {hidden} more distinct character{'s' if hidden != 1 else ''}" if hidden else ""
    noun = "character" if len(offenders) == 1 else "characters"
    return UnsupportedScriptError(
        "text contains characters outside the supported alphabets "
        f"(English, Russian, Kazakh): {details}{more} ({len(offenders)} offending {noun})",
        offenders=tuple(offenders),
    )
def validate_article_text(text: str, *, max_length: int = DEFAULT_MAX_LENGTH) -> str:
    """Gate raw input; return NFC text or raise one documented exception."""
    # 1. type
    if not isinstance(text, str):
        raise TypeError(f"text must be str, not {type(text).__name__}")
    # 2. config sanity (programmer error, not an article error)
    if isinstance(max_length, bool) or not isinstance(max_length, int) or max_length < 0:
        raise ValueError(f"max_length must be a non-negative int, not {max_length!r}")
    # 3. size
    if len(text) > max_length:
        raise TextTooLongError(
            f"article text is too long: {len(text)} characters exceeds "
            f"the limit of {max_length}",
            length=len(text), max_length=max_length,
        )
    # 4. NUL
    nul = text.find("\x00")
    if nul != -1:
        raise MalformedUnicodeError(
            f"article text contains a NUL character (U+0000) at index {nul}",
            kind="nul", index=nul,
        )
    # 5. lone surrogate
    m = _SURROGATE.search(text)
    if m:
        cp = ord(m.group())
        raise MalformedUnicodeError(
            f"article text contains a lone surrogate (U+{cp:04X}) at index {m.start()}",
            kind="surrogate", index=m.start(),
        )
    # 6. foreign script
    nfc = unicodedata.normalize("NFC", text)
    offenders, first_seen = _scan_script(nfc)
    if offenders:
        raise _unsupported_error(offenders, first_seen)
    # 7. nothing to count
    if not _HAS_ALLOWED.search(nfc):
        raise EmptyArticleTextError(
            "article text contains no supported letters (English, Russian, Kazakh)"
        )
    return nfc
    
```

## 10. Integration guidance

**HTTP mapping** (recommended, so clients get actionable codes):

|Exception|Status|Client-facing message|
|---|---|---|
|`TextTooLongError`|`413 Payload Too Large`|article exceeds the size limit|
|`MalformedUnicodeError`|`400 Bad Request`|article text is not valid Unicode|
|`UnsupportedScriptError`|`422 Unprocessable Entity`|article contains unsupported characters|
|`EmptyArticleTextError`|`422 Unprocessable Entity`|article contains no countable words|
|`TypeError`, config `ValueError`|`500 Internal Server Error`|_(do not expose)_|

**Logging:** on rejection, log exception class plus structured fields — `exc.kind`, `exc.index`, `exc.length`, `exc.offenders` (characters and code points only). **Never log the raw article text**: it is user content, may be large, and may contain the very characters you are rejecting.

**Batch jobs:** one bad article must not abort the run — catch `InvalidArticleText`, record the article id with the exception class, continue. Do **not** catch `TypeError`; that is a code bug.

## 11. Relationship to `normalize` (defense in depth)

- `normalize` is total and never raises on validated input (normalization §2). The script check lives exclusively in `validate_article_text`.
    
- `normalize` never raises `UnsupportedScriptError`. If foreign letters reach `normalize` in production, it means **`validate_article_text` was skipped** and they were silently stripped — a wiring bug, not user error. Treat it as an alertable event and add a test/lint rule that bans direct `normalize(` calls outside the pipeline module.
    
- The two functions share the same letter set (§3 and normalization §3). A property test should assert invariant §7.3 (`normalize(validate_article_text(t)) == normalize(t)`) over a corpus of mixed-script, emoji, digit, punctuation, and NFD strings, and invariant §7.2 (no `normalize` exception after successful validation).
    
- This gate does **not** change v2's definition of a word. Removing it would restore v2's behavior exactly; adding it only rejects input earlier, with better diagnostics.
