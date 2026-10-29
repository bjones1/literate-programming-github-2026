# Top Words — Specification

## 1. What the function should do

A news website needs to show the most frequently used words in an article. The `top_words` function reads the article text, counts how many times each word appears, and returns the ten most common words.

The function must support English, Russian, and Kazakh text. Uppercase and lowercase versions of the same word must be treated as identical.

## 2. Function requirements

The function must be written in Python 3 and stored in `top_words.py`.

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

The function takes one string and returns a list of tuples. Each tuple contains a word and its frequency. The result must contain no more than ten distinct words.

## 3. How words are counted

To keep the results consistent, the function follows these rules:

- Uppercase and lowercase letters are treated as the same. For example, `Toyota` and `TOYOTA` count as one word.
- A word is a sequence of one or more Unicode alphabetic characters. Python's `isalpha()` method determines which characters are letters.
- Spaces, punctuation marks, digits, hyphens, and apostrophes separate words.
- Punctuation is not part of a word. For example, `BMW, BMW.` counts as two occurrences of `bmw`.
- A hyphen separates words. For example, `Mercedes-Benz` is counted as `mercedes` and `benz`.
- Apostrophes also separate words. For example, `driver's` is counted as `driver` and `s`.
- Before counting, each word is converted to lowercase using Python's `lower()` method.
- The function does not remove common words, translate words, or change them to their base forms.
- Words are counted according to their spelling after lowercasing. No stemming or lemmatization is performed.

## 4. Ordering the results

The function must rank words by frequency, from highest to lowest.

If several words have the same frequency, they must be sorted alphabetically according to Python's string comparison order. This rule makes the output predictable regardless of the order in which words appear in the article.

If the article contains fewer than ten distinct words, the function returns all of them. If it contains no words, the function returns an empty list.

The function must count all distinct words before applying the ten-word limit.

## 5. Examples

The following examples define the expected behavior of the function.

| Input | Expected output |
|---|---|
| `Toyota BMW Toyota Audi BMW Toyota` | `[('toyota', 3), ('bmw', 2), ('audi', 1)]` |
| `BMW bmw BMW Mercedes MERCEDES` | `[('bmw', 3), ('mercedes', 2)]` |
| `Toyota, Toyota. (Toyota)! BMW?` | `[('toyota', 3), ('bmw', 1)]` |
| `Mercedes-Benz BMW Mercedes-Benz` | `[('benz', 2), ('mercedes', 2), ('bmw', 1)]` |
| `Toyota Honda Ford` | `[('ford', 1), ('honda', 1), ('toyota', 1)]` |
| `Audi Audi BMW BMW Mercedes` | `[('audi', 2), ('bmw', 2), ('mercedes', 1)]` |
| `Toyota Toyota BMW Honda Ford Audi Kia Hyundai Nissan Mazda Volvo Subaru` | `[('toyota', 2), ('audi', 1), ('bmw', 1), ('ford', 1), ('honda', 1), ('hyundai', 1), ('kia', 1), ('mazda', 1), ('nissan', 1), ('subaru', 1)]` |
| `Toyota 123 BMW 456 Toyota` | `[('toyota', 2), ('bmw', 1)]` |
| `""` | `[]` |
| `"   !!! ... "` | `[]` |
| `Toyota Toyota Honda Honda BMW` | `[('honda', 2), ('toyota', 2), ('bmw', 1)]` |

## 6. Input limitations

The function is designed to receive a Python string. Behavior for other input types, such as integers, lists, or `None`, is outside the scope of this specification.

The function must not print anything, ask the user for input, or modify the original string.

## 7. Completion criteria

The implementation is considered correct when it:

1. Counts words according to the rules in this document.
2. Treats uppercase and lowercase versions as the same word.
3. Supports English, Russian, and Kazakh letters.
4. Sorts words by frequency and applies the specified tie-breaking rule.
5. Returns no more than ten unique words.
6. Produces exactly the expected output for every example in Section 5.

## 8. Clarifications for implementation

- Punctuation separates words; it is not removed to join neighboring text.
- Hyphens and apostrophes separate words. For example, `rock-roll` produces `rock` and `roll`, while `can't` produces `can` and `t`.
- Digits are separators and are not counted as words.
- Ties are resolved by ascending Python string order, not by first appearance.
- The ten-word limit is applied after counting and sorting all distinct words.