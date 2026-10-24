# LLM review

**Model:** Claude Sonnet 5.5, extra high effort
**Prompt:** `Review spec.md` (with the current `spec.md` attached)

Strikethrough indicates LLM output that I deemed incorrect or irrelevant.

## Output

**Summary:** No significant issues. All 35 examples are consistent with rules 1-10, no rule contradicts another, and I found no input in the intended domain (news articles in English, Russian, Kazakh) where two reasonable implementations would disagree on the main behavior. Only minor gaps remain.

**Minor gaps (non-blocking)**

1. **Letter-hyphen-digit tokens.** Under rules 3 and 4, `COVID-19` is one candidate token (the hyphen sits between `D` and `1`), and it is discarded whole, so `covid` is not counted. This is consistent but has no example. Add `COVID-19 covid` → `[("covid",1)]`.
2. **`isdigit()` boundaries.** `²` is a digit (`m²` is dropped), while `½` and `Ⅷ` are neither digits nor letters and act as separators (`a½b` → `a`, `b`). This follows from the rules but may surprise a reader; one example would pin it down.
3. **Format characters.** Soft hyphen (U+00AD) and ZWJ/ZWNJ are not letters or marks, so they split a word. Copy-pasted news text sometimes contains soft hyphens. State whether splitting is intended.
4. **Casefold expansion of `İ`.** `İ` casefolds to `i` + U+0307, so `İstanbul` and `istanbul` are different words. This follows from rule 1 but is not shown in an example.
5. **Example 25** depends on the reader noticing that the first letter is Latin. Write it with an escape (`a\u0431`), as examples 19-21 do.
6. **Out-of-scope wording.** "Performance beyond linear time" is unclear, since sorting the distinct words is not linear. Say "no performance requirements".
7. ~~**Stop words.** Consider removing stop words, because a trending box dominated by "the" and "и" is not useful.~~

**Verdict:** The spec is unambiguous enough to produce one plausible implementation. Items 1-6 are optional clarifications.