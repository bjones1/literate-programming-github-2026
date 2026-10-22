# Specification: Top Words

## Purpose

The purpose of this program is to identify the ten most frequent words in a text.

## Function interface

Implement a Python 3 function named `top_words` that accepts a single string argument, `text`, and returns a list of `(word, count)` tuples.

Each tuple contains a normalized word and the number of times it occurs in the input.

## Word definition

1. A word is a maximal consecutive sequence of Unicode letters.
2. English, Russian, and Kazakh letters are supported.
3. Whitespace, punctuation, digits, and symbols separate words and are not included in the result.
4. Combining marks are not considered letters under this specification.

## Normalization and counting

1. Convert each word to lowercase using Python's `str.casefold()` method.
2. Count all occurrences of each normalized word.
3. Do not remove stop words or perform stemming or lemmatization.
4. Treat words that become identical after case folding as the same word.

## Sorting and output

1. Sort words by decreasing frequency.
2. For words with equal frequencies, sort by ascending Unicode code-point order of the normalized word.
3. Return at most ten distinct words.
4. If fewer than ten distinct words occur, return all of them.
5. If the input contains no words, return an empty list.
6. Return a list of tuples, not a dictionary or formatted string.

## Examples

- Input: `cat dog cat bird dog cat`
  Output: `[("cat", 3), ("dog", 2), ("bird", 1)]`
- Input: `Cat cat CAT`
  Output: `[("cat", 3)]`
- Input: `hello, world! hello.`
  Output: `[("hello", 2), ("world", 1)]`
- Input: `привет мир привет`
  Output: `[("привет", 2), ("мир", 1)]`
- Input: `қазақ тілі қазақ`
  Output: `[("қазақ", 2), ("тілі", 1)]`
- Input: `123 !!! ...`
  Output: `[]`
- Input: `a b c d e f g h i j k`
  Output: `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]`

## Assumptions

- The input is a Python string.
- The function does not need to read files or interact with the user.
- The function must produce deterministic output for the same input.
- The implementation language is Python 3.