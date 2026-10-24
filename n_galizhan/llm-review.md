Prompting Claude Opus 5.5 (`claude-opus-5-5`), run as a Claude Code subagent with file and shell tools at its default effort, with the prompt below produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant. Notes marked **Response** are mine and say what I changed in `spec.md` afterwards.

This was the seventh review. Each review ran in a fresh context with the same prompt, except for the scratch directory, and I fixed each review's findings before running the next. Row numbers in the output refer to the version the review read.

**Prompt:**

```text
Review spec.md

The file is /Users/fine/literate-programming-github-2026/n_galizhan/spec.md (review only this file). It is a student's specification for this exercise from a course on writing specifications:

> A news site shows a "Trending words" box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh. Python 3. Name the function `top_words` and put it in `top_words.py`.
>
> Tighten this specification to produce one plausible result. Then, for each decision your spec makes, describe at least one input and its expected output, usually as a table in your spec. A useful example gets a different output under some other plausible reading of the original spec.

Find anything that would let two careful implementers build different programs: ambiguities, contradictions, factual errors, examples whose expected output does not follow from the rules, and missing decisions that matter for English, Russian or Kazakh news text. Also flag text that is redundant or longer than it needs to be. Rank findings by severity and be concrete (quote the line, give an input that exposes the problem). Do not rewrite the spec.

Do not modify any files in the repository. If you write code to check the examples, put it in /private/tmp/claude-501/-Users-fine-literate-programming-github-2026/03ab780c-66c2-44ec-8e99-5c5c439622f7/scratchpad/review7/ and run Python with `python3.14 -I` (3.11 is also installed as `python3.11`). Your final message is the review itself, in Markdown, addressed to the spec's author.
```

**Output:**

---

# Review of `n_galizhan/spec.md`

## What I checked

I wrote a literal implementation of your eight steps and ran every table row, including the subclass, fresh-list and `TypeError` rows, on CPython 3.11.15 (Unicode 14.0) and 3.14.8 (Unicode 16.0). **All 27 rows produce exactly the output you give.**

I also checked these factual claims, and all of them are correct:
- The Unicode categories you cite: U+2139 is `Ll`, U+FE0F is `Mn`, U+02BB and U+02BC are `Lm`, U+200B is `Cf`, U+00B4 and U+0060 are `Sk`, U+2116 is `So`, and U+00B2 and U+00BD are `No`.
- `[^\W\d_]+` keeps `км²` and `½`.
- The NFKC result for `№`.
- The Unicode version of each Python release.
- The only existing character whose properties differ between Unicode 14.0 and 16.0 is U+1171E, which moves from `Mn` to `Mc`. Both start with `M`, so this changes nothing.
- The Kazakh 2021 Latin mapping: `I ı` = `І і`, `İ i` = `И и`/`Й й`, `Y y` = `Ы ы`. Your parenthetical is right.

I found **no ambiguity, contradiction or wrong example** that would let two careful implementers diverge. Step 4 had one possible reading gap: does a "run of letters" include joiners, as a Step 5 word does? The `20th-century` → `century` row settles it. What follows are consequences of your decisions that matter for news text, one sentence that could mislead, and text you can cut.

## Findings, by severity

### Medium

**1. Step 4's 1-to-3-letter rule drops real words and keeps real suffixes.**
The rule reads: "a numeral suffix is a run of one to three letters right after a digit, or after a digit and one `'` or `-`, that is not followed by another letter". It is unambiguous, but length cannot tell a suffix from a word:

| Input | Output under your rules | Problem |
| --- | --- | --- |
| `"a 10-day ceasefire and a 3-day summit"` | `[("a", 2), ("and", 1), ("ceasefire", 1), ("summit", 1)]` | `day` is never counted |
| `"5-year-old 5-day-old"` | `[("old", 1), ("year-old", 1)]` | The same construction gives different words |
| `"2-for-1 deal, 3-way talks"` | `for` and `way` disappear | 3-letter words lost |
| `"Қазанның 24-інде және 25-інде"` | `[("інде", 2), ("және", 1), ("қазанның", 1)]` | A Kazakh date ending becomes the top word, while `24-і` would be dropped |
| `"100км 100 км 5kg 5 kg"` | `[("kg", 1), ("км", 1)]` | A unit counts only when spaced. Row 8 shows only the spaced form |
| `"1–ші 1-ші 1—ші"` | `[("ші", 2)]` | An en or em dash before a suffix leaves it counted |

Row 16's "Rules out" says you do not drop "every word that follows a number". You do drop every 1-to-3-letter word that follows one. You have two options:
- Keep the rule, say so, and add a row such as `10-day`.
- Replace the length test with a list of suffixes.

> **Response:** Addressed by narrowing the rule rather than replacing it. Step 4 now drops only one- or two-letter runs (`th`, `s`, `х`, `го`, `ші`, `ке`), so `day`, `for`, `way` and `day-old` are counted, and the `10-day` example in the table shows it. I accept the remaining costs in this table: `інде`, `нан`, units written without a space, and a suffix after an en dash. Step 4 names `нан` as one of them. A suffix list covering three languages would be long and still incomplete.

### Low

**2. Step 1's "This is equivalent to Unicode's canonical caseless matching" invites a different program.**
- As a matching relation the sentence is true, because NFC(x) = NFC(y) exactly when NFD(x) = NFD(y).
- But the standard (D145) is written as `NFD(toCasefold(NFD(X)))`. A reader who implements that form gets NFD output, and your Step 2 then strips the diaeresis of `ё`, the breve of `й` and every Kazakh-Latin diacritic. For `"ёж й Öskemen"`, your formula gives `[("öskemen",1),("й",1),("ёж",1)]`; the NFD form gives `[("oskemen",1),("еж",1),("и",1)]`.
- Your formula is explicit, so the sentence only gives a reader reason to doubt it. Cut it.
- The Turkic sentence repeats row 7. It also understates the cost: every all-caps Kazakh-Latin word with `І` is affected. For example, `"BIR bır"` → `[("bir",1),("bır",1)]`, and `bir` is the spelling of a different word, one written with `и`.

> **Response:** Addressed. The canonical-caseless sentence is gone. The Turkic sentence now states the general cost: any Kazakh-Latin word with a capital `I` misses its lowercase form. I kept its one clause of reason (English `In`), because the row cannot show why the cost is accepted.

**3. Common news-text results that the rules decide but no example shows.**
None of these is ambiguous, but a reader judging the spec against the "Trending words" purpose will ask whether you meant them. One row would show they are deliberate.
- `"U.S. officials said the U.S. will"` → `u` and `s` are the top two words (2 each).
- `"В 2024 г. рост составил 5 млрд руб., т.е. больше"` → `г`, `т` and `е` are counted as words.
- ~~`"US troops told us"` → `[("us", 2), …]`, and `"WHO said who"` → `("who", 2)`. Casefolding merges acronyms with ordinary words.~~
- `"Подробнее: https://www.inform.kz/ru/news"` → `https`, `www`, `kz` and `ru` are counted as words.

> **Response:** Addressed with one new row, `"U.S. т.е. https://inform.kz"`, which shows that abbreviations and URLs split like any other text. The struck bullet is intended behavior: merging `US` with `us` is what case-insensitive counting means, and row 2 already shows it.

**4. Redundant clause in Step 4: "together with a `'` or `-` right after it".**
It has no effect. Once the suffix becomes a space, the joiner after it has no letter before it, so Step 5 treats it as a separator anyway. I compared the word lists with and without the clause on 300,000 random strings built from `a b 1 ' -` and space, and they never differed. Applying Step 4 repeatedly instead of once made no difference either. This is the only clause with no example, because no example could show it. Delete it.

> **Response:** Addressed; the clause is deleted, and my reference implementation gives the same results without it.

**5. Redundant examples.**
- Row 18 (`"catкот"`) rules out "splitting where the script changes". Row 3's `бiрiншi` is already one word mixing Cyrillic and Latin letters, so row 3 rules that out too.
- ~~Row 5's third alternative ("deleting a grave accent that NFC composes into `ѐ`") is the same reading that row 4 rules out ("deleting marks before NFC").~~

> **Response:** The `catкот` row is removed, and row 3's "Rules out" now names script changes. I kept the grave-accent example: it also shows that a grave stress mark survives NFC, which Step 2's acute-only example does not make obvious, and an earlier review asked for that row.

~~**6. The table does not say which step each row tests.**~~
~~With 27 rows, a reader checking that every decision has an example has to rebuild the mapping. A step column, or rows grouped under step headings, would make coverage easy to check. It would also have exposed finding 4.~~

> **Response:** Not changed. Each row's "Rules out" cell already names the decision it pins down, and a fourth column would widen a table that is already wide.

---
The scripts are in /private/tmp/claude-501/-Users-fine-literate-programming-github-2026/03ab780c-66c2-44ec-8e99-5c5c439622f7/scratchpad/review7/:
- `top_words.py`: the literal implementation
- `check.py`: runs all table rows
- `probe.py` and `probe2.py`: the inputs above
- `fuzz.py`: the Step 4 comparison
- `cmp.py`: the Unicode 14 vs 16 comparison

No repository files were modified.
