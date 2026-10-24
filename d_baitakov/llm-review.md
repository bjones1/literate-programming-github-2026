# LLM review of `spec.md`

**LLM used:** Claude Code (CLI agent) running GLM (glm-5.3-flash, Z.ai),
in an interactive terminal session, at default (medium) reasoning effort.

**Prompt used:** `Review the revised spec.md for the Top words problem. Check
every definition for ambiguity, every table row for a wrong expected output,
and every requirement from the instructor's review for actually being met.
For each finding, say whether a plausible alternative reading of the original
problem would produce a different output.`

This file shows the last review performed (of the revised spec, after the
instructor's change requests were applied; it supersedes the earlier review).
Strikethrough indicates review output I deemed incorrect or irrelevant and
did not act on.

---

## Review output

1. **Invisible combining dots in an output cell.** The `"İNİ ini"` row's
   expected output contains `i` + U+0307 twice, which a reader cannot see
   and which an NFC-normalizing editor could alter. The instructor's feedback
   asked for escapes on *inputs*; the same problem exists in this *output*.
   → **Accepted.** The output is now written as
   `[("ini", 1), ("i̇ni̇", 1)]`.

2. **Tie order in the `"rock'’n"` row.** The first draft of the row
   listed `[("rock", 1), ("n", 1)]`, but by code point `n` (U+006E) sorts
   before `rock` (U+0072). → **Accepted, fixed.** Correct order is
   `[("n", 1), ("rock", 1)]`. (Found by running all 22 rows through a
   reference implementation written directly from the definitions — an
   `isalpha()`-based scanner, not the `[^\W\d_]+` regex shortcut, which the
   table's `km²` row shows would be wrong.)

3. **Kazakh Latin limitation actually pinned.** Checked that the two
   required rows exist and match the implementation: `QIZ qız` gives two
   entries (casefold maps `I` to `i`, not `ı`) and `"İNİ ini"` gives two
   entries (`İ` casefolds to `i` + U+0307). → **Confirmed met.**

4. **Apostrophe run rule now decidable.** The word definition says a
   *single* apostrophe between two letters joins, and the `don''t` and
   `rock'’n` rows pin both the doubled and the mixed-script cases.
   `don't't` follows from the rule (the words `don't` and `t`) without
   needing its own row. → **Confirmed met; no change.**

5. **NFC status no longer reads as a precondition.** The "assumed to be in
   NFC" sentence is gone; the behavior for decomposed input is defined
   directly (combining marks are not letters, so they separate words) and
   pinned by the `"éclair éclair"` row. → **Confirmed met.**

6. ~~The spec should also cover Turkish and Azerbaijani, whose alphabets
   share the `I/ı/İ/i` dotted-dotless pairing.~~ **Rejected as out of
   scope.** The problem names English, Russian, and Kazakh only; the Kazakh
   limitation is documented because the problem claims Kazakh support.

7. ~~Counts should exclude single-letter words, since a trending-words box
   full of "a" and "и" is not useful.~~ **Rejected.** No such rule exists in
   the original problem, and adding one would trade one plausible reading
   for another rather than leaving one.

## Verification

After acting on findings 1–2, all 22 rows of the examples table were run
against the reference implementation. 22/22 match their expected output.
