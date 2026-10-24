# LLM Review of `spec.md`

This supersedes the previous `llm-review.md`. The instructor's own review
(Claude Opus 5.5, medium effort) on this PR required changes to the prior
spec — a wrong example in decision 5, a real contradiction between the word
definition's prose (`isalpha()`) and its regex (which also matched
non-decimal numerics like `км²`), and several Cyrillic edge cases
(typographic apostrophe/hyphen variants, combining stress marks) that split
one visible word into two. All of that is fixed in the current `spec.md`,
using the corrected tokenization pipeline from that review, which I
re-verified independently (every table row, old and new, checked against a
fresh run of the implementation — see the commit for details).

What follows is the next review on top of that fix: another fresh Claude
Sonnet 5 subagent, no memory of writing or fixing the spec, so this is a
genuine independent pass rather than the same context re-approving its own
corrections.

Prompting Claude Sonnet 5 (via Claude Code, fresh subagent) with:

> You are reviewing a specification for a university assignment (CSDA 5101,
> Advanced Software Paradigms). Read the file at this path: [spec.md]. This
> is a REVISED version of a spec, after the instructor gave a detailed
> critical review of an earlier draft (not shown to you — you have no
> memory of the earlier draft and should review this version purely on its
> own merits). Context: the original, under-specified problem statement
> this spec is tightening is: "A news site shows a 'Trending words' box
> beside each article. Write a Python function that finds the 10 most
> common words in an article's text, along with how many times each one
> appears. Articles may be in English, Russian, or Kazakh." The goal of the
> spec is to leave only ONE plausible implementation. Review spec.md
> against that original prompt. Specifically check: 1) Multilingual
> correctness (English/Russian/Kazakh, including Cyrillic edge cases like
> combining marks, typographic punctuation variants, and non-decimal
> Unicode numerics). 2) Tie-breaking and ranking — fully deterministic and
> internally consistent (does the prose match the stated sort order)? 3)
> Edge cases — empty input, fewer than 10 words, non-string input. 4)
> Internal consistency — does every example table row's "expected output"
> actually follow from the rules and code given earlier in the document?
> Check specifically for ordering correctness in multi-word tied-count
> examples (don't just trust alphabetization by eye — Cyrillic code-point
> order is easy to get wrong by hand). 5) Anything still ambiguous, or
> anything missing that the original prompt implies but the spec never
> addresses. Do not read or run top_words.py, check_tables.py, or
> verify_spec_tables.py in that directory — review spec.md on its own
> merits. Give a numbered list of findings, real gap or nitpick. Be
> genuinely critical.

produced the following output; strikethrough indicates LLM output that I
deemed incorrect or irrelevant.

---

I hand-traced the `_words()` pipeline and `str.casefold()`/sort rules
against every example row in all 7 decisions ... Every expected output is
arithmetically correct and internally consistent with the stated rules —
this is a notably careful spec. Findings:

1. **Real, minor — Cf-stripping/ZWSP inconsistency.** Step 2 deletes Mn/Cf
   characters outright rather than replacing them with a space. The stated
   rationale is "these ... still shouldn't silently split one visible word
   into two." But zero-width space (U+200B, category Cf) is itself a word
   *separator* in its one real-world use; deleting it instead of blanking
   it does the opposite of the stated goal — it silently *merges* two
   visible, separate words into one.

   **Applied.** Verified empirically (`top_words("foo​bar")` gave one
   word, `foobar`, before the fix). Reordered the pipeline to fold
   U+200B to a space in the same step as the other typographic
   substitutions, before the Mn/Cf-drop step runs, so it now acts as a
   separator instead of vanishing.

2. **Real, moderate — overgeneralized claim in decision 5.** "Code-point
   order puts ё (U+0451) after every other Cyrillic letter" is false within
   this spec's own trilingual domain: Kazakh letters ә(04D9), ғ(0493),
   қ(049B), ң(04A3), ө(04E9), ұ(04B1), һ(04BB) all have *higher* code
   points than ё, so ё sorts *before* them, not after "every" Cyrillic
   letter.

   **Applied.** Verified empirically (`ord('ё')` is 0x451; all nine
   Kazakh-specific letters are 0x456 and above). Reworded to state
   precisely where `ё` falls: after plain Russian `а`–`я`, but before the
   nine Kazakh-specific letters.

3. ~~**Nitpick — structural inconsistency.** The "Kazakh is Cyrillic"
   resolution lives in the Interface section, not as its own numbered
   Decision with an input/output/alternate-reading table like every other
   resolved ambiguity.~~

   **Rejected.** This is a scoping assumption stated up front (like "Python
   3" or the function's name), not a point where two careful readers of the
   *original prompt* would disagree the way, say, case-folding or
   tie-breaking invite disagreement. Promoting it to a full decision would
   add a table and more prose for a single uncontested sentence, working
   against the instructor's own note that this spec is already too long for
   one function.

4. **Nitpick — imprecise wording.** Decision 5's sentence "the first 10 of
   the tied words ... are kept" is only literally true when the tie spans
   the whole top-10 band; in a mixed case it should say "the first N
   remaining slots."

   **Applied.** Reworded to describe the actual algorithm (sort the full
   list by count then code point; take the top 10 of that one sorted list),
   which is correct in both the all-tied and partially-tied cases, instead
   of a description that only covered one of them.

5. **Nitpick — asymmetric example coverage.** Doubled hyphen is tested
   ("well--known") but doubled/leading/trailing apostrophe is not.

   **Applied**, minimally: added a one-clause note on the existing
   `"well--known"` row rather than a new row, to avoid adding another full
   table entry for a case the regex already handles symmetrically (verified
   with `top_words("don''known")` → `don: 1, known: 1`, same shape as the
   hyphen case).

No missing boundary cases (empty/whitespace/digits-only input, <10 distinct
words, non-str/bytes/None input) — all fully and correctly specified.

---

After applying findings 1, 2, 4, and 5, I re-ran every example in every
table against a fresh implementation two different ways: a hand-maintained
test file, and a script that scrapes the literal input/output text directly
out of `spec.md`'s own tables and checks it against `top_words()` — so the
committed document is checked against real code, not just my memory of
writing it. Both pass on every row. (One input — the decomposed-`й` example
— is written as a Python expression rather than a literal pasted string,
because decomposed Unicode does not reliably survive being pasted as plain
text through this document's own editing pipeline; I confirmed this the
hard way, twice, before switching to that representation.)
