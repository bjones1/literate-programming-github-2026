LLM Review
==========

---

Model, Prompt
-------------

---

Prompting **OpenAI GPT-6**, default effort, with `Review spec.md`, strikethrough
should indicate LLM output that is deemed incorrect or irrelevant. This is the
second review performed on `spec.md` that was updated for eight rounds with new
fixes provided

Prompt
------

---

Review file `spec.md`. It is a specification of this problem

Implement a Python function that analyzes an article written in English,
Russian, or Kazakh and returns its 10 most frequent words, along with their
occurrence counts.

The function must:

* Count words without distinguishing uppercase and lowercase forms.
* Support English, Russian, and Kazakh letters, including Kazakh-specific
  letters.
* Handle Unicode text safely and consistently.
* Define explicit behavior for punctuation, digits, whitespace, invisible
  characters, and malformed or unexpected inputs.
* Return deterministic results, including when multiple words have the same
  frequency.

The function does not need to determine the article's language. It applies the
same Unicode-based rules to every input.

Output
------

---

Review of `top_words` Specification
-----------------------------------

Executive summary
-----------------

The specification is detailed and its core behavior is generally coherent: it
defines input validation, Unicode normalization, token boundaries, case folding,
counting, ordering, and the ten-item limit.

This review identifies additional edge cases and clarity issues. Most are
missing clarifications or test gaps rather than contradictions in the core
algorithm. The most important fixes are to correct the Markdown code fence and
clarify that Unicode character categories---not visual appearance---determine
whether a character belongs to a token.

Findings
--------

### 1\. The output example may leave a Markdown code fence unclosed

**Severity: Medium**

In Section 13, the Python example appears to open a fenced code block without
closing it after the example output. If the closing fence is missing in the
actual file, Markdown renderers may treat the remainder of the document as code.

<strong>Recommendation:</strong> Add a closing triple-backtick fence immediately
after the example output.

### 2\. Unicode category rules can conflict with character names or visual appearance

**Severity: Medium**

The specification says that token characters are determined by Unicode general
categories. It also identifies straight and curly apostrophes as separators.
This is compatible with category-based rules, but other visually or semantically
apostrophe-like characters may fall into accepted categories.

For example, U+02BC MODIFIER LETTER APOSTROPHE has category `Lm` and would
therefore be accepted as a token character under the stated category rule,
unlike `'` or `’` if those are explicitly treated as separators.

<strong>Recommendation:</strong> State explicitly that the listed
Unicode-category rules take precedence over a character's name, appearance, or
linguistic role. If U+02BC is intended to be a separator too, add an explicit
exception and corresponding test.

### 3\. Variation selectors may be included in tokens

**Severity: Low**

Variation selectors commonly have the Unicode category `Mn`. Under the current
rules, they may continue an existing token and affect the resulting key even if
the visual difference is subtle or invisible. For example, `a` and `a\uFE0F` may
be counted as different words.

<strong>Recommendation:</strong> Clarify that category rules apply even to
invisible or presentation-related combining marks, and add a test involving
U+FE0F. The implementation should not compare words by visual appearance.

### 4\. The specification should explicitly rule out retokenization after case folding

**Severity: Low**

The specification says that each completed token is case-folded and then
normalized to NFC. It would be helpful to state explicitly that transformed
tokens are not tokenized again. This matters because case folding can expand a
character; for example, `ß` case-folds to `ss`.

<strong>Recommendation:</strong> Add a sentence such as: "After case folding and
NFC normalization, the resulting string is used as one counting key; it is not
tokenized again."

### 5\. The acceptance tests do not cover several category boundaries

**Severity: Medium**

The current tests could be expanded to demonstrate the intended behavior at
important Unicode boundaries. Useful cases include:

* A letter in category `Lm` adjacent to other letters.
* A combining mark at the beginning of input, where it must not start a token.
* Combining marks following letters and digits.
* Variation selectors such as U+FE0F.
* Unicode line and paragraph separators.
* Additional format characters (`Cf`) beyond the explicitly named examples.
* Isolated surrogate code points (`Cs`), which should act as separators under
  the category-based rules.
* Input containing only combining marks.

<strong>Recommendation:</strong> Add explicit expected outputs for these cases.
Tests should verify the category-based rule, not merely the appearance of the
sample text.

### 6\. The order of NFC normalization and tokenization should be highlighted

**Severity: Medium**

The specification requires NFC normalization of the whole input before
tokenization. That ordering can affect token boundaries and which characters are
present when the token rules are applied.

For example, `a\u0301` is normally composed into `á` by NFC, while a leading
combining mark in `\u0301a` remains before the letter and cannot start a token
under the stated rules.

<strong>Recommendation:</strong> Emphasize that tokenization is performed on the
whole-input NFC-normalized string, not on the original input. Include both
examples in the tests.

### 7\. Unicode whitespace coverage could be more explicit

**Severity: Low**

The specification broadly treats whitespace and separators as token delimiters,
but examples can help ensure implementations do not rely only on ASCII
whitespace or `str.isspace()`.

Consider adding cases for:

* U+00A0 NO-BREAK SPACE
* U+2003 EM SPACE
* U+2028 LINE SEPARATOR
* U+2029 PARAGRAPH SEPARATOR
* U+0085 NEXT LINE

<strong>Recommendation:</strong> Include representative examples from multiple
Unicode categories and state that the defined category/token rules govern the
outcome.

### 8\. Add a permutation-invariance test

**Severity: Low**

For the same multiset of tokens, changing the order of the input should not
change the final result, provided the token counts are unchanged. This is a
useful check for counting and tie-breaking behavior.

For example, both `b a b a c` and `a b c a b` should produce:

```python
[("a", 2), ("b", 2), ("c", 1)]
```

<strong>Recommendation:</strong> Add this as a regression test.

### 9\. Separate normative requirements from examples and tests

**Severity: Low**

A specification is easier to implement consistently when mandatory behavior,
illustrative examples, and executable acceptance tests are clearly
distinguished. A checklist is helpful, but it does not replace test cases with
exact inputs and expected outputs.

<strong>Recommendation:</strong> Keep the normative rules authoritative, label
examples as illustrative, and maintain a separate acceptance-test table or test
section with exact expected outputs.

Priority summary
----------------

---

\# Finding Severity Suggested priority

---

1 Potentially unclosed Medium Fix first Markdown code fence

2 Unicode category Medium Fix first precedence
for<br>apostrophe-like<br>characters

3 Variation selectors may Low Add clarification be part of tokens and test

4 No explicit prohibition Low Add clarification on retokenizing<br>transformed
tokens

5 Missing Unicode Medium Expand test suite category-boundary tests

6 NFC-before-tokenization Medium Clarify and test behavior needs emphasis

7 More Unicode whitespace Low Expand test suite examples needed

8 Missing Low Add regression permutation-invariance test test

9 Normative rules, Low Improve examples, and tests could organization be
separated
------------------------------------------------------------------------

Overall assessment
------------------

No new contradiction was found in the core counting, normalization, sorting, or
top-ten behavior. The findings above primarily concern underspecified edge
cases, test coverage, and document presentation.

<strong>Recommended order of work:</strong>

1. Fix the Markdown code fence.
2. Clarify the precedence of Unicode-category rules over visual or linguistic
   interpretation.
3. Make the NFC-before-tokenization and no-retokenization rules explicit.
4. Expand the acceptance tests for Unicode edge cases and deterministic
   ordering.

---

<p><br></p>
