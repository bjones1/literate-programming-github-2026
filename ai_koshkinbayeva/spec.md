# Top Words Specification

## Purpose

Implement a Python 3 function named `top_words(text)` that returns the most frequent words in a text.

## Input

- `text` must be a string.
- The text may contain English, Russian, and Kazakh words.
- An empty string is a valid input.

## Output

- Return a list of `(word, count)` tuples.
- Each word is lowercase.
- Each count is a positive integer.
- Return at most 10 distinct words.

## Word definition

- A word is a consecutive sequence of Unicode letters.
- Words are case-insensitive.
- Punctuation, digits, whitespace, and underscores separate words.
- Apostrophes and hyphens separate words.
- Do not remove stop words or apply stemming or lemmatization.

## Counting and ordering

- Count every occurrence of each word.
- Sort by frequency in descending order.
- If two words have the same frequency, sort them alphabetically by Unicode code-point order.
- If fewer than 10 distinct words exist, return all of them.
- If no words are present, return an empty list.

## Errors

- If `text` is not a string, raise `TypeError`.

## Examples

| Input | Expected output |
|---|---|
| `"cat dog cat"` | `[("cat", 2), ("dog", 1)]` |
| `"Cat CAT cat"` | `[("cat", 3)]` |
| `"hello мир әлем hello"` | `[("hello", 2), ("әлем", 1), ("мир", 1)]` |
| `""` | `[]` |
| `"123 !!! _"` | `[]` |
| `"pear apple pear apple"` | `[("apple", 2), ("pear", 2)]` |

## Scope

The function processes the supplied string only. File reading, HTML processing, translation, stemming, and lemmatization are out of scope.