Prompting ChatGPT-5.6 Luna, with `Review spec.md` produced the following output;

## Review of `top_words` Specification

The specification is comprehensive, well organized, and easy to follow. It establishes a clear processing pipeline and provides precise rules for validation, Unicode normalization, tokenization, stopword filtering, and sorting. The reference tests cover a wide range of important cases and help make the expected behavior unambiguous.

**Minor suggestions for improvement:**
1. **Test coverage:** Add explicit tests for unequal frequencies, Unicode character categories, and surrogate characters in string subclasses.
2. **Edge cases:** Include a few more tests for combining marks and consecutive separators to strengthen validation of tokenization rules.

Overall, this is a strong specification with clearly defined behavior and good test coverage. The suggested changes are minor and would make it even more consistent and reliable.