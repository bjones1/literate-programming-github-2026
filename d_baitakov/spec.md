# Specification: `top_words`

## Problem

A news site shows a "Trending words" box beside each article. Write a Python 3
function `top_words` in `top_words.py` that finds the 10 most common words in
an article's text, along with how many times each one appears. Articles may be
in English, Russian, or Kazakh.

## Definitions

**Input.** `top_words(text: str) -> list[tuple[str, int]]`: a single string of
already-extracted article text. Passing anything other than `str` is a caller
error the function need not handle.

**Word.** A maximal run of letters (characters for which `str.isalpha()` is
true), in which a single `'` (U+0027) or `’` (U+2019) between two letters
joins them into one word and is kept. Every other character separates words.
In particular digits, hyphens, and underscores split words, and a run of two
apostrophes does not join (`don''t` is the two words `don` and `t`).

**Output.** A list of `(word, count)` tuples, at most 10, where a word's
count is the number of words in `text` whose casefolded form equals it and
the `word` in the output is that casefolded form. The list is ordered by
count, highest first; words with equal counts are ordered lexicographically
by code point. The same tie-break decides the cut: if the 10th and 11th
words have equal counts, the lexicographically smaller one is kept. If there
are fewer than 10 distinct words, all are returned; if there are none, the
result is `[]`. The limit 10 is fixed by the problem statement, not a
parameter.

**Case.** Each word is normalized with `str.casefold()` before counting, so
`Cat`, `cat`, and `CAT` are one word. Known limitation for Kazakh written in
the 2021 Latin alphabet: `casefold()` maps `I` to `i` (not `ı`) and `İ` to
`i` followed by U+0307, so a capitalized word can fail to match its
lowercase form; the two table rows below pin this behavior. No other
normalization is performed: `ё` ≠ `е`, `café` ≠ `cafe`, and combining marks
are not letters, so a decomposed `e` + U+0301 + `clair` is the two words
`e` and `clair` while NFC `éclair` is one word. There is no stop-word
filtering: with no list in the problem, `the`, `и`, and `және` count like
any other word.

## Examples

Unquoted inputs are shown as plain text; quoted inputs are Python string
literals (used where invisible code points matter).

| Input | Expected output |
|---|---|
| `cat dog cat bird dog cat` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| `Cat cat CAT` | `[("cat", 3)]` — case is folded |
| `Straße STRASSE` | `[("strasse", 2)]` — `lower()` would give two words |
| `QIZ qız` | `[("qiz", 1), ("qız", 1)]` — the documented casefold limitation |
| `"İNİ ini"` | `[("ini", 1), ("i̇ni̇", 1)]` — `İ` casefolds to `i` + U+0307 (shown escaped in the output) |
| `100 km² km` | `[("km", 2)]` — `²` is not a letter (`isalpha()` false) |
| `covid19 covid covid` | `[("covid", 3)]` — digits split; `19` is not a word |
| `state-of-the-art` | `[("art", 1), ("of", 1), ("state", 1), ("the", 1)]` — hyphens split; 1–1 ties break by code point |
| `foo_bar foo` | `[("foo", 2), ("bar", 1)]` — underscore splits |
| `"don''t"` | `[("don", 1), ("t", 1)]` — a run of two apostrophes does not join |
| `"rock'’n"` | `[("n", 1), ("rock", 1)]` — each apostrophe must sit between two letters |
| `don't don't dont` | `[("don't", 2), ("dont", 1)]` — internal apostrophe joins; `don't` ≠ `dont` |
| `"don't don’t"` | `[("don't", 1), ("don’t", 1)]` — U+0027 and U+2019 are different joining characters |
| `'tis 'tis` | `[("tis", 2)]` — a leading apostrophe does not join |
| `42 42 ---` | `[]` — no words at all |
| `Кот кот КОТ` | `[("кот", 3)]` — Cyrillic casefolds too |
| `ёж ёж еж` | `[("ёж", 2), ("еж", 1)]` — `ё` is not folded to `е` |
| `Мен мен және Мен` | `[("мен", 3), ("және", 1)]` — no stop-word removal |
| `"éclair éclair"` | `[("clair", 1), ("e", 1), ("éclair", 1)]` — no NFC normalization; combining marks split words |
| `b a c a b c d` | `[("a", 2), ("b", 2), ("c", 2), ("d", 1)]` — equal counts sort lexicographically |
| `k j i h g f e d c b a` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` — eleven 1-count words tie at the cut; `k` is dropped because `j < k` |
| `""` | `[]` |
