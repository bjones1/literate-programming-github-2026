# Specification: top\_words

## Purpose:

`top_words` returns up to `n` most frequent words in a piece of plain text, along with their frequency counts. Supports English, Russian, Kazakh, and multilingual text. The function is language-agnostic and relies strictly on Unicode properties and deterministic rules.

## Interface:

File: `top_words.py`, Python 3\. Signature: `def top_words(text: str, n: int = 10) -> list[tuple[str, int]]:`

## Processing Pipeline:

The text processing pipeline executes in the following strict order:

1. Input Validation & Initial NFC Normalization: Validate types and normalize input text to Unicode Normalization Form C (NFC).  
2. Character Preprocessing: Map hyphen and apostrophe variants to standard ASCII equivalents (`-` and `'`), and delete soft hyphens (`U+00AD`) and format characters (`Cf`).  
3. Tokenization: Extract maximal letter runs containing allowed internal single joiners.  
4. Post-Tokenization Normalization: Lowercase tokens using `str.lower()` followed by secondary NFC normalization.  
5. Counting & Filtering: Count frequencies, discard pure-digit tokens, sort by frequency (descending) then Unicode codepoint (ascending), and slice up to `n`.

## Rules & Validation:

1. Parameter `text`: Must be a `str`. If not (including `None`), raises `TypeError`. Checked first.  
2. Parameter `n`:  
   - Defaults to `10`. Must be a non-negative strict integer.  
   - Type validation: Must satisfy `isinstance(n, int) and not isinstance(n, bool)`; otherwise raises `TypeError` (runs before negativity check, e.g., `n=-1.5` or `n=True` raises `TypeError`).  
   - Value validation: If `n < 0`, raises `ValueError`.  
   - `n = 0` returns `[]`.  
3. Return Value: Returns up to `n` pairs of `(word, count)`, ordered by frequency (descending), then by Unicode codepoint order (ascending) for ties. If `n` exceeds the number of distinct words, all distinct words are returned.  
4. Empty or Non-Word Text: Empty strings `""`, whitespace-only strings, or text containing only digits/punctuation return `[]`.  
5. Side Effects: Must not modify its input and must not print anything.  
6. Dependencies: Pure standard library (`unicodedata`, `re`).

## Implementation Details & Edge Cases:

1. Unicode Letters & Marks:  
   - Letters are defined by Unicode General Categories `L` (`Lu`, `Ll`, `Lt`, `Lm`, `Lo`) and marks by category `M` (`Mn`, `Mc`, `Me`).  
   - Combining marks (`Mn`, `Mc`, `Me`) attach to preceding base letters. A combining mark at the start of text or following a separator without a preceding base letter is treated as part of the separator/ignored.  
   - Russian stress marks (e.g., combining acute accent `U+0301` in `замо́к`) remain attached under NFC and count as distinct tokens from unaccented words.  
2. Punctuation Mapping & Deletion:  
   - Typographic apostrophes (`‘`, `’`, `ʼ`, `ʻ`, `` ` ``) are mapped to standard straight apostrophe (`'`).  
   - Hyphen variants (`U+2010` HYPHEN, `U+2011` NON-BREAKING HYPHEN, `U+2012` FIGURE HYPHEN, `U+2013` EN DASH, `U+2014` EM DASH, `U+2015` HORIZONTAL BAR) are mapped to standard ASCII hyphen (`-`).  
   - Soft hyphen (`U+00AD`) and all General Category `Cf` characters (e.g., zero-width joiner `U+200C`, zero-width non-joiner `U+200D`, word joiner `U+2060`) are deleted prior to tokenization.  
3. Word Definition & Boundaries:  
   - Maximal runs of letters and combining marks.  
   - Single internal apostrophes (`'`) or hyphens (`-`) join letter runs (e.g., `don't`, `state-of-the-art`).  
   - Mixed joiner sequences (e.g., `-'`, `'-'`) or multiple consecutive punctuation marks (e.g., `co--op`, `don''t`) act as separators, splitting words.  
   - Leading or trailing hyphens and apostrophes are stripped (e.g., `-word` yields `word`).  
   - Digits, underscores, whitespace, and punctuation act as word separators. `abc123def` yields `abc` and `def`. Pure-digit tokens (e.g., `123`) and words like `mp3` yield `mp` (as digits are stripped/separated).  
4. Case Conversion & Normalization:  
   - Tokens are lowercased using standard Python `str.lower()`, followed by an NFC re-normalization step. (Note: `ß` remains `ß`, and `İ` follows Python standard lowercasing).  
5. Special Characters (`ё` and `е`):  
   - `е` (`U+0435`) and `ё` (`U+0451`) are kept distinct. `е` sorts before `ё` in Unicode codepoint order.  
6. Other Scripts:  
   - CJK and Thai scripts without spaces form single multi-character tokens (entire sentences/phrases), which is standard for language-agnostic tokenization.

## Examples Table:

| Input Text | n | Output | Description / Edge Case |
| :---- | :---: | :---- | :---- |
| `"Hello world hello"` | 10 | `[('hello', 2), ('world', 1)]` | Basic frequency count |
| `"don't co--op"` | 10 | `[('co', 1), ('dont', 1), ('op', 1)]` | Single apostrophe join vs consecutive hyphens |
| `"'tis students' state-of-the-art"` | 10 | `[('state-of-the-art', 1), ('students', 1), ('tis', 1)]` | Leading apostrophe, trailing apostrophe, hyphenated compound |
| `"rock-'n'-roll"` | 10 | `[('n', 1), ('rock', 1), ('roll', 1)]` | Mixed joiner sequences acting as separators |
| `"İstanbul"` | 10 | `[('i̇stanbul', 1)]` | Capital dotted I lowercased |
| `"замо́к замок"` | 10 | `[('замок', 1), ('замо́к', 1)]` | Russian stress mark distinctness |
| `"hy‐phen"` | 10 | `[('hy-phen', 1)]` | Non-ASCII hyphen variant mapping |
| `"hyp­hen"` | 10 | `[('hyphen', 1)]` | Soft hyphen deletion |
| `"mp3 track"` | 10 | `[('mp', 1), ('track', 1)]` | Digits acting as separators |
| `"12345"` | 10 | `[]` | Pure-digit token ignored |
| `""` | 10 | `[]` | Empty string |
| `"   "` | 10 | `[]` | Whitespace-only string |
| `"a b a b c"` | 2 | `[('a', 2), ('b', 2)]` | Truncation to n=2 |
| `"abc"` | 0 | `[]` | n=0 returns empty list |
| `"café café"` | 10 | `[('café', 2)]` | NFC normalization unity |

&nbsp;