# Specification: Top Words

## 1. Purpose and scope

The function `top_words` finds the 10 most frequent distinct words in an article and returns each word together with its frequency.

The specification is designed for text in which words are separated by spaces or punctuation, such as English, Russian, and Kazakh, and, within the limits stated in Section 3, other scripts such as Devanagari. It contains no language-specific rules: for example, there is no Turkish-specific case mapping and no special handling of zero-width joiners. Text in languages written without spaces between words (for example Chinese, Japanese, and Thai) is accepted, but the function performs no word segmentation: every uninterrupted run of letters is a single word (see the example in Section 8).

## 2. Function interface

- Language: Python 3.
- Function name: `top_words`.
- Input: one Python string containing the article text (see Section 9 for what counts as a string).
- Return value: a list of `(word, frequency)` tuples, as defined in Section 7.
- The function has no side effects and does not change any shared state.

## 3. Definition of a word

A *letter* is a character whose Unicode General Category begins with `L` (`Lu`, `Ll`, `Lt`, `Lm`, `Lo`). A *combining mark* is a character whose General Category begins with `M` (`Mn`, `Mc`, `Me`).

A **word** is a maximal, non-empty sequence of characters in which every letter may be followed by any number of combining marks. In regular-expression notation: `(\p{L}\p{M}*)+`. This notation only describes the rule; Python's standard `re` module does not support `\p{...}`. If the prose and the notation ever disagree, the notation decides.

The word definition is applied to the text after the normalization and case-folding steps of Section 4. Whether a character is a letter or a combining mark is therefore judged on the folded text. For example, U+0345 (combining ypogegrammeni) is a combining mark, but it case-folds to `ι` (U+03B9), which is a letter, so the text `"\u0345"` contains the word `ι`.

Consequences:

- A combining mark belongs to a word only if it directly follows a letter or another combining mark that belongs to that word. A combining mark at the start of the text, or after a character that is not a letter or mark, is a separator.
- Every other character separates words. Separation depends only on the General Category of a character, not on what the character looks like or means. In particular:
  - Whitespace, punctuation, hyphens, dashes, and underscores separate words.
  - The ASCII apostrophe `'` (U+0027) and the right single quotation mark `’` (U+2019) separate words. The modifier letter apostrophe `ʼ` (U+02BC) has category `Lm`, so it is a letter and does not separate words: `can't` gives `can` and `t`, but `canʼt` is one word.
  - Digits separate words and are not included in words.
  - Number characters that are not decimal digits, such as `²` (superscript two) and `Ⅻ` (Roman numeral twelve), also separate words, because their General Category is not `L`.
  - Format characters (category `Cf`) such as the soft hyphen U+00AD, the zero-width joiner U+200D, and the zero-width non-joiner U+200C separate words. Words in some scripts that use these characters are therefore split at them; this is intentional.
  - Control, surrogate, private-use, and unassigned characters separate words. Which code points are unassigned depends on the Unicode version (see the last paragraph of Section 4).
- A string containing no letters produces no words.

For example, `a`, `Я`, `ә`, `қ`, `ң`, `ө`, `ұ`, `ү`, `і`, `ғ`, and `һ` are letters. In the Hindi word `नमस्ते`, the virama `्` and the vowel sign `े` are combining marks, so the whole word is one word.

## 4. Unicode normalization and case

The function performs these operations in order, on the whole input, before splitting it into words:

1. Normalize the input to Unicode NFC.
2. Apply Python's `str.casefold()`.
3. Normalize the result to Unicode NFC again.

NFC is used, not NFKC. NFC does not fold compatibility characters; only the foldings defined by Unicode full case folding are applied. Therefore the ligature `ﬁ` (U+FB01) and the letters `fi` are the same word, because `casefold()` turns `ﬁ` into `fi`, but full-width `ａ` (U+FF41) and `a` (U+0061) are different words, because case-folding does not change `ａ`. This is intentional.

Case-folding is not the same as lowercasing. For example, `ß` folds to `ss`, so `Straße` and `STRASSE` are the same word. Case-folding can also produce combining marks: `İ` (U+0130) folds to `i` followed by U+0307 (combining dot above). No single precomposed character exists for this sequence, so the result stays two code points. Because a word may contain combining marks (Section 3), `İstanbul` is a single word: `i` + U+0307 + `stanbul`. The combining mark is kept; it is not removed. Consequently `İstanbul` and `istanbul` are different words.

Words that are identical after these operations are the same word. For example, `Apple`, `APPLE`, and `apple` are all counted as `apple`.

The word returned in the result is the normalized, case-folded form, not the original spelling. The folded form is not always lowercase: for example, the Cherokee letter U+AB70 folds to the uppercase-form letter U+13A0.

The Unicode properties (General Category, case-folding, normalization) are those of the Unicode database of the Python interpreter that runs the function.

## 5. Counting

Each occurrence of a word contributes exactly one to that word's frequency. Repeated occurrences are counted independently.

## 6. Ranking and selection

The function ranks the distinct words as follows:

1. Words with higher frequencies come first.
2. If two words have the same frequency, they are ordered by **Unicode code-point order**: the words are compared character by character, by code point, and the first difference decides. A word comes before any longer word that starts with it, so `can` comes before `cant`. This is the same as Python's comparison of `str` values with `<`.
3. After sorting all distinct words, the function keeps only the first 10.

Code-point order is intentional. It is not alphabetical order for every language. For example, the Kazakh letter `ә` (U+04D9) comes after `я` (U+044F) in code-point order, although it comes right after `а` in the Kazakh alphabet.

The tie-break (rule 2) is applied before the limit of 10 (rule 3). It therefore determines which words are included when several words are tied for the last available positions.

## 7. Output

The return value is a list of `(word, frequency)` tuples.

- The list contains at most 10 tuples.
- Each word is a plain `str` and appears at most once.
- Each frequency is a positive `int`.
- The tuples are sorted according to Section 6.
- If the input contains fewer than 10 distinct words, all of them are returned.
- If the input contains no words, the function returns an empty list: `[]`.
- The function returns a list even when it contains zero or one tuple.

## 8. Examples and expected outputs

All inputs in the table are Python expressions: string literals (so escape sequences such as `\t`, `\n`, and `\u0301` have their usual Python meaning), plus `None`, a `bytes` literal, and a `collections.UserString` instance for the error cases. Unless a row says "raises `TypeError`", the expected output is a Python list of tuples.

| Input | Expected output | Rule tested |
|---|---|---|
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | Frequency counting |
| `"Apple apple APPLE"` | `[("apple", 3)]` | Case-insensitive matching |
| `"cat, dog! cat."` | `[("cat", 2), ("dog", 1)]` | Punctuation separates words |
| `"cat can't cant"` | `[("can", 1), ("cant", 1), ("cat", 1), ("t", 1)]` | Apostrophes separate words; ties use code-point order; a prefix comes first (`can` < `cant`) |
| `"well-known well known"` | `[("known", 2), ("well", 2)]` | Hyphens separate words; ties use code-point order |
| `"cat123dog cat 123 dog"` | `[("cat", 2), ("dog", 2)]` | Digits separate words |
| `"cat_dog cat dog"` | `[("cat", 2), ("dog", 2)]` | Underscores separate words |
| `"x²y Ⅻ"` | `[("x", 1), ("y", 1)]` | Number characters that are not digits separate words and are not part of words |
| `"Ана ана Бала"` | `[("ана", 2), ("бала", 1)]` | Cyrillic letters and case-folding |
| `"Қазақ қазақ Тіл"` | `[("қазақ", 2), ("тіл", 1)]` | Kazakh letters and case-folding |
| `"b a c"` | `[("a", 1), ("b", 1), ("c", 1)]` | Code-point order when frequencies tie |
| `"я ә а"` | `[("а", 1), ("я", 1), ("ә", 1)]` | Code-point order, not the Kazakh alphabet order (`ә` is last) |
| `"cafe\u0301 café"` | `[("café", 2)]` | NFC: `e` + U+0301 and `é` (U+00E9) are the same word; the result uses U+00E9 |
| `"Straße STRASSE"` | `[("strasse", 2)]` | Case-folding is not lowercasing |
| `"ﬁne fine"` | `[("fine", 2)]` | `casefold()` folds the ligature `ﬁ` (U+FB01) to `fi` |
| `"\u0130stanbul"` | `[("i\u0307stanbul", 1)]` | `İ` folds to `i` + U+0307; the combining mark stays inside one word |
| `"नमस्ते दुनिया"` | `[("दुनिया", 1), ("नमस्ते", 1)]` | Combining marks are part of words (Hindi) |
| `"\u0345"` | `[("ι", 1)]` | Letter and mark status is judged after case-folding: U+0345 is a mark that folds to the letter `ι` |
| `"co\u00adoperate"` | `[("co", 1), ("operate", 1)]` | A format character (soft hyphen, category `Cf`) separates words |
| `"can\u2019t can\u02bct"` | `[("can", 1), ("can\u02bct", 1), ("t", 1)]` | U+2019 separates words; the modifier letter U+02BC is a letter |
| `"a\ud800a"` | `[("a", 2)]` | A lone surrogate separates words |
| `"\uab70"` | `[("\u13a0", 1)]` | The folded word is not always lowercase |
| `"\u0301a 1\u0301b"` | `[("a", 1), ("b", 1)]` | A combining mark that does not follow a letter is a separator |
| `"ａ a"` | `[("a", 1), ("ａ", 1)]` | NFC, not NFKC: full-width `ａ` (U+FF41) differs from `a` |
| `"日本語のテキスト"` | `[("日本語のテキスト", 1)]` | No word segmentation for languages written without spaces |
| `"z z a b c d e f g h i j k"` | `[("z", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` | Top-10 limit; `j` and `k` are excluded |
| `"z z z a a b b c c d d e e f f g g h h i i j j k k"` | `[("z", 3), ("a", 2), ("b", 2), ("c", 2), ("d", 2), ("e", 2), ("f", 2), ("g", 2), ("h", 2), ("i", 2)]` | Tie at the 10th place with frequencies above 1: `a` to `k` all have frequency 2 and compete for nine places, so `i` is the last word that fits; ties are broken before the cut, so `j` and `k` are excluded |
| `"cat dog"` | `[("cat", 1), ("dog", 1)]` | Fewer than 10 distinct words |
| `""` | `[]` | Empty input |
| `"   \t\n "` | `[]` | Whitespace-only input |
| `"123 ! @#$"` | `[]` | Input with no letters |
| `None` | raises `TypeError` | Invalid input type |
| `b"cat"` | raises `TypeError` | `bytes` is not a string |
| `UserString("cat")` | raises `TypeError` | `collections.UserString` is not a subclass of `str` |

## 9. Preconditions and errors

The input must be an instance of Python's `str` type. The check is `isinstance(text, str)`, so instances of subclasses of `str` are accepted; they are processed by their string value, as if their characters were copied into a plain `str`. Any other argument, including `None`, `bytes`, `bytearray`, `collections.UserString`, and numbers, makes the function raise `TypeError`.

No other input restrictions apply. An empty string and a string containing only separators are valid inputs and return an empty list.

## 10. Conformance requirement

An implementation conforms to this specification only if it follows the stated normalization, word definition, counting, ranking, selection, output-format, and error-handling rules.

Each example in Section 8 must produce exactly the specified output.