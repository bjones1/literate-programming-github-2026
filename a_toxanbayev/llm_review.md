# LLM Review

**Model:** GPT-6  
**Prompt:** Review the provided `spec.md` for the Top words problem. Identify ambiguities, missing requirements, inconsistencies, and edge cases. Focus on whether the specification is clear, complete, and testable. Suggest specific improvements without rewriting the entire specification.

## Review Output

The specification provides a basic outline of the `top_words` function, including its name, programming language, input type, output, and an example. However, several important details are missing or ambiguous.

### 1. Function Requirements

- The specification states that `top_words` must be a global function, but it does not explain whether this means a module-level function or whether any additional restrictions apply.
- It does not specify whether the function should return a dictionary, a list, or a formatted string. The output description (`word: count of word appearances -> int`) is ambiguous.
- The function's behavior for empty input is not defined.

### 2. Input and Text Processing

- The input is described as `input_sentence -> str`, but it is unclear whether the function accepts only a single sentence or arbitrary text containing multiple sentences.
- The specification does not define how words are separated. It is unclear whether whitespace alone separates words or whether punctuation should also be treated as a separator.
- Case sensitivity is not specified. For example, should `Cat`, `cat`, and `CAT` be counted as the same word or as three distinct words?
- The handling of punctuation is undefined. For example, should `cat`, `cat,`, and `cat.` be counted as the same word?
- The specification does not clarify how numbers, apostrophes, hyphens, or other non-letter characters should be handled.

### 3. Language Support

- The statement "Articles may be in English, Russian and Kazakh" is ambiguous. The word "articles" could refer to grammatical articles, written texts, or something else.
- If the intention is to support texts written in English, Russian, and Kazakh, the specification should state this explicitly.
- It is unclear whether the function must recognize language-specific characters, including Cyrillic letters and Kazakh-specific characters.
- The specification does not explain whether words from different languages should be processed together using the same counting rules.

### 4. Output Requirements

- The output description does not define the exact return type or structure.
- The example shows words and their counts in a formatted string, but it does not establish whether this is the required output format or merely a human-readable representation.
- The ordering of words is unspecified. It is unclear whether results should be sorted alphabetically, by descending frequency, or by order of first appearance.
- The behavior when multiple words have the same frequency is not defined.

### 5. Definition of "Top Words"

- The function is named `top_words`, but the specification asks for the number of appearances of each word.
- It is unclear whether the function should return the counts of all unique words or only the most frequent words.
- If only the most frequent words are required, the specification does not define how many words to return or how ties should be handled.

### 6. Example and Edge Cases

The provided example is useful, but it does not cover important cases, such as:

- Empty input.
- Input containing only whitespace.
- Repeated words with different capitalization.
- Words followed by punctuation.
- Text containing English, Russian, and Kazakh characters.
- Multiple words with equal frequencies.

Additional examples would make the specification easier to implement and test.

## Recommendations

To improve the specification:

1. Define the exact return type, preferably a dictionary mapping each word to its occurrence count, if that matches the intended requirements.
2. Clarify whether all unique words or only the most frequent words should be returned.
3. Define case sensitivity and punctuation handling.
4. Clarify whether the function supports arbitrary text or only a single sentence.
5. Replace "Articles may be in English, Russian and Kazakh" with an explicit statement about supported languages.
6. Specify how words are separated and how numbers and special characters are handled.
7. Define the ordering of results, if ordering matters.
8. Include examples covering empty input, punctuation, capitalization, and multilingual text.

## Overall Assessment

The specification is a reasonable starting point, but it is not yet sufficiently precise to guarantee consistent implementations. The most important issues are the ambiguity in the output format, the undefined meaning of "top words," and the lack of explicit text-processing rules. Resolving these points would make the specification more complete, unambiguous, and testable.