# Review of spec.md

Prompting Claude Opus 5.5, extra high effort, with `Review spec.md` (Claude
Code, `claude -p "Review spec.md" --model claude-opus-5-5 --effort xhigh`, with
read-only file tools) produced the following output; strikethrough indicates
LLM output that I deemed incorrect or irrelevant, and each **Note** says why.
This is the last review, run on the `spec.md` in this commit.

The review could not run code, so I ran every table row against a reference
implementation of section 2 on CPython 3.13; all 37 rows pass.

> I found no errors in `spec.md`. I couldn't run Python here because the command needed approval that isn't available in this session. Instead I worked through all 37 table rows by hand against section 2, including the three rows this diff adds or changes, and every row gives the expected output. Please re-run your reference implementation anyway, since the rows have changed since your last run.
>
> ## What I checked
>
> - **New row 63, `"\uff15G \u0663"`:** `５` is `isdecimal()`, so the token is `５G`. NFD doesn't touch the fullwidth digit, which agrees with row 92 ("no NFKC"), so the word is `\uff15g`. `٣` alone has no letter, so it's dropped. Code that only accepts `0`–`9` would give `[("g", 1)]`, so the row catches that mistake.
> - **Changed row 64, `"x\u00b2 \u00bdx XXI"`:** The old input had `½` standing alone, where it was dropped whether or not it counted as a digit, so the row didn't test it. Now `½x` would become one word under `isnumeric()`/`isalnum()`, and `x²` under `isdigit()`. The row now catches all three wrong choices.
> - **Changed row 71, ZWJ added:** U+200D is `Cf`, so it's deleted and `newspaper` gets 3. This also matches UAX #29, which ignores ZWJ (rule WB4) but breaks at ZWSP.
> - **Line 35, "left after step 1":** This fixes a real contradiction. Before, deleted `Cf` characters were also "every other character", so they counted as separators.
> - **Step 1 list:** The extra deleted characters match the Default_Ignorable_Code_Point characters outside `Cf` in Unicode 16.0 (checked from memory).
> - **Literal characters:** Every literal `’ ‐ ‑ ‘ ʼ − — ² ½ ﬁ İ` is the code point its text says it is. No word mixes Latin and Cyrillic letters. There are no literal invisible characters or combining marks. The single-letter `е`, `а` (lines 96, 112) are Cyrillic, and `і` (line 97) is U+0456.
>
> ## Should fix
>
> ~~1. **`llm-review.md` no longer matches the spec.** This diff applies the reviewer's items 4, 5 and 6, but the review file still says:~~
>    - ~~items 4 and 5 were "Left as is";~~
>    - ~~item 6 is struck through, with a note rejecting it;~~
>    - ~~"all 36 rows pass", but there are now 37 rows, and rows 64 and 71 have changed.~~
>
>    ~~Update those notes to "Fixed" and update the row count after you re-run. If this review is going to replace the old one in `llm-review.md`, that also takes care of it.~~

**Note:** This is about the previous `llm-review.md`, not the spec. This review replaces it, as the item itself suggests, and I re-ran all 37 rows.

> ## Minor
>
> 2. **`spec.md:23–24`: the Latin-script Kazakh wording is still a little narrow.** Words that only ever contain a lowercase `i` are affected too. For example, `"İä iä"` gives two entries, `i̇ä` and `iä`, and the word `iä` contains none of `I`, `ı` or `İ`. A more exact version: "words with a dotted or dotless `i`, in either case, may be miscounted".

**Note:** Correct, but it only refines the wording of a case the spec already declares unsupported; the behavior is fully defined by section 2 and by the `İstanbul` row.

> ~~3. **`spec.md:63`: `\u0663` adds nothing to the test.** Every plausible implementation drops it. That includes ASCII-only, `isnumeric` and `isalnum` versions, and anything that got it wrong would already fail row 61. Keep it only if you want to show a digit from a non-Latin script; it doesn't hurt.~~

**Note:** It is there on purpose, to show a digit from a non-Latin script, which the review says is fine.

> ~~4. **`spec.md:41`: list order.** U+FFA0 comes before U+FE00–U+FE0F. Every other entry is in code-point order, so move U+FFA0 after them.~~

**Note:** Cosmetic; the order of the list has no effect.

> Nothing else needs changing. The procedure still leaves no room for two correct implementations to disagree, and the new rows fill the gaps they were meant to.
