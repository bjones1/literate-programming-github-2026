# LLM review of spec.md

Prompting **Claude Opus 5.5** (claude.ai, model ID `claude-opus-5-5`) with
`Review spec.md` produced the following output. This is the last review run;
it was run on the version of `spec.md` just before the final commit. Earlier
reviews found missing handling for combining stress marks (U+0301), Unicode
hyphens (U+2010, U+2011), the modifier-letter apostrophe (U+02BC), and
superscript digits; those are now in the spec.

~~Strikethrough~~ indicates LLM output that I deemed incorrect or irrelevant;
each one has a note explaining why.

As a final check, Claude wrote `top_words.py` from the spec; it produced every
expected output in the spec's tables (27 of 27).

---

## Review output

**1. Lowercasing can reintroduce combining marks.** Step 1 removes `Mn`
characters before step 4 lowercases. But `"İ".lower()` returns `i` followed by
U+0307 COMBINING DOT ABOVE, so `"İnternet"` and `"internet"` would count as
different words. This matters for Kazakh in Latin script, which uses `İ`.
Either remove `Mn` characters again after lowering, or lowercase before step 1.

> **Fixed:** step 4 now deletes `Mn` characters produced by lowering, with the
> example `"İnternet internet"` → `[("internet", 2)]`.

**2. Kazakh Latin `I`/`ı` pairing is unspecified.** In Kazakh Latin script,
uppercase `I` pairs with dotless `ı`, but Python's `str.lower()` maps `I` to
`i`. State which behavior you want.

> **Fixed:** step 4 now says lowering is locale-independent (`I` → `i`), with
> the example `"Iı"` → `[("iı", 1)]`.

~~**3. Consider excluding stop words.** A "Trending words" box full of `the`,
`и`, and `мен` is not useful. Add stop-word lists for English, Russian, and
Kazakh.~~

> **Rejected:** the spec deliberately has no stop-word list (step 5) and shows
> it with an example. Choosing word lists for three languages would add a large
> second source of ambiguity; this is a product decision, not a gap.

~~**4. Consider `str.casefold()` instead of `str.lower()`.** `casefold()` is
the recommended method for caseless matching.~~

> **Rejected:** for English, Russian, and Kazakh text the two differ only on
> characters like German `ß`, which these articles don't use. The spec already
> names `str.lower()` exactly, so there is only one plausible result.

~~**5. Returning a `dict` may be more convenient than a list of tuples.**~~

> **Rejected:** a list of tuples preserves the ranking order, which the box
> needs. The return type is already fixed in the Interface section.

~~**6. Specify performance limits for very large articles.**~~

> **Irrelevant:** the assignment asks for one plausible *result*; running time
> doesn't change the output.

**7. The step 1 `"й й"` example is ambiguous when read on screen,** since both
forms render identically.

> **Addressed:** the row already says which `й` is decomposed (`и` + U+0306);
> no change needed beyond that note.

**No other errors or underspecified behaviors found.** Input type, word
boundaries (letters, apostrophes, hyphens, digits, dashes), normalization,
case, counting, tie-breaking, and truncation each have an exact rule and at
least one example that distinguishes it from another plausible reading.
