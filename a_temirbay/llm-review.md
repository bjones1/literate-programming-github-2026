Prompting Gemini 3, medium effort, with `Review spec.md` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

# LLM Review for `spec.md`

## Review Metadata
- **LLM Used:** Gemini 3
- **Effort Level:** Medium
- **Prompt Used:** `Review spec.md`

---

## LLM Review Output

Here is a comprehensive review of the `top_words` specification (`spec.md`).

Overall, the specification is **exceptionally thorough, clear, and well-structured**. It anticipates edge cases that frequently trip up text-processing pipelines (such as Unicode normalization forms, soft hyphens, combining marks, and Python's boolean subclassing behavior for integers).

---

### Key Strengths

1. **Rigorous Parameter Validation:**
* Explicitly handling `n`'s type validation (`isinstance(n, int) and not isinstance(n, bool)`) prevents subtle bugs where booleans (which are subclasses of `int` in Python) could pass standard type checks.
* The ordering of checks (type validation before negativity checks) is precisely defined.


2. **Robust Unicode & Normalization Strategy:**
* Enforcing Unicode Normalization Form C (NFC) at both the input and post-tokenization stages ensures consistent character representation (e.g., combining accents and base characters are harmonized).
* Detailed categorization using Unicode General Categories (`L`, `M`, `Cf`) makes the tokenization rules language-agnostic rather than relying on brittle ASCII-only character classes.


3. **Precise Handling of Punctuation & Joiners:**
* Clear rules for internal single joiners (`'` and `-`) versus mixed or consecutive punctuation (`-'`, `co--op`), ensuring correct word boundary detection (e.g., distinguishing `state-of-the-art` from `rock-'n'-roll`).
* Explicit mapping tables for non-standard apostrophes and hyphen variants eliminate ambiguity during preprocessing.


4. **Comprehensive Examples:**
* The provided examples cover standard behavior, tricky edge cases (stress marks, pure digits, soft hyphens, empty/whitespace strings), and tie-breaking criteria (descending frequency, ascending Unicode codepoint order).



---

### Implementation Considerations & Suggestions

While the specification is complete and ready for implementation, keep the following technical nuances in mind during coding:

* **Standard Library Limitations (`re` vs `regex` module):**
* Rule 6 specifies a pure standard library dependency (`unicodedata`, `re`). Python’s built-in `re` module has limited built-in support for full Unicode property escapes (like `\p{L}` or `\p{M}`) depending on the Python version, or requires careful use of character classes/UNICODE flags. Developers should ensure the regex engine accurately matches Unicode categories `L` and `M` using standard library features or explicitly defined Unicode property ranges if `\p` is unavailable.


* **Tie-Breaking Performance:**
* For the sorting step (*"sort by frequency (descending) then Unicode codepoint (ascending)"*), ensure that Python's Timsort stability is leveraged correctly by sorting by Unicode codepoint first (or using a multi-key sort tuple like `key=lambda x: (-x[1], x[0]))`, which cleanly handles both constraints in a single sort pass.



### Conclusion

The specification is well-crafted, logically sound, and leaves very little room for ambiguity. It provides a solid blueprint for building a robust, language-agnostic word frequency counter.