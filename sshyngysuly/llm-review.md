# LLM Review of `spec.md`

## Model used

- Model: OpenAI GPT-6
- Review type: Specification review
- Effort setting: Not specified

## Prompt used

Review the specification below for the Top Words programming problem.

Identify:

1. Ambiguous or underspecified requirements.
2. Contradictions between requirements and examples.
3. Missing edge cases.
4. Potential implementation or testing problems.

Do not rewrite the entire specification. Give specific, actionable feedback and explain why each issue matters. If the specification is already clear, say so.

Evaluate the specification as a contract for a Python 3 function named `top_words`.

The complete contents of `spec.md` were provided after this prompt.

## Last review output

The specification was reviewed for ambiguity, consistency, missing edge cases, and implementation and testing concerns.

The review identified the following issues:

1. **Unicode letter definition:** The specification does not clearly state whether letters from languages other than English, Russian, and Kazakh are allowed. Python's `str.isalpha()` recognizes letters from many Unicode scripts. The specification should explicitly define the intended scope.

2. **Case normalization terminology:** The specification says to convert words to lowercase using `str.casefold()`. Case folding is not identical to ordinary lowercasing. The contract should state that `casefold()` is the required normalization operation.

3. **Combining characters and Unicode normalization:** A character such as an accented letter can have different Unicode representations. The current definition excludes combining marks, which can cause visually equivalent text to be tokenized differently. The specification should state whether Unicode normalization, such as NFC, is required.

4. **Missing edge-case examples:** Additional tests should cover empty strings, whitespace-only input, ties at the top-ten boundary, and words containing apostrophes or hyphens. These examples would clarify tokenization and selection behavior.

5. **Tie-breaking tests:** The existing example with single-letter words supports the specified Unicode ordering. An additional example with words presented in a different input order would help verify that ties are resolved independently of input order.

6. **Invalid input types:** The function contract assumes a string but does not specify behavior for other argument types. The specification should clarify whether non-string inputs are outside the contract or should raise a particular exception.

## Conclusion

The specification defines the function interface, counting rules, result limit, and deterministic ordering. The review found no direct contradictions between the provided examples and the stated requirements. The main improvements would be to clarify Unicode handling and add tests for edge cases.

