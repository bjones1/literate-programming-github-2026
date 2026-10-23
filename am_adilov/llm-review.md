# LLM review of `spec.md`

Prompting **Claude Opus 5** (1M-context variant, model id `claude-opus-5[1m]`,
run as a Claude Code subagent at default effort) with the prompt quoted below
produced the following output; strikethrough would indicate LLM output that I
deemed incorrect or irrelevant, and this review contains none — it reports no
findings. What I rejected in earlier rounds is recorded under
[Round history](#round-history).

The reviewer was a fresh instance with no knowledge of how the spec was
written. It was allowed to run `python3` to check code points, casefolding and
sort order, and to implement the rules and compare against the examples, but
not to edit the spec.

**This is the last review performed. Its verdict is that the spec has no
significant issues, and it reviewed `spec.md` exactly as it stands in this
commit** — nothing in the spec changed after it ran. It is review round 5;
rounds 1 through 4 raised 18, 11, 3 and 0 significant findings respectively,
each addressed before the next round ran.

## The prompt

> Review the specification at
> `/Users/adilovamir/kbtu/csda/literate-programming-github-2026/am_adilov/spec.md`:
> find errors, ambiguities, underspecified behavior, and places where the
> examples contradict the rules.
>
> The spec answers this problem statement from a course e-book:
>
> "A news site shows a 'Trending words' box beside each article. Write a Python
> function that finds the 10 most common words in an article's text, along with
> how many times each one appears. Articles may be in English, Russian, or
> Kazakh. Python 3. Name the function top_words and put it in top_words.py."
>
> This is a later review round; earlier reviews have already been addressed, so
> expect the spec to be tight. Judge it as a specification an implementer must
> be able to follow to exactly one program.
>
> Read the spec file, then write your review as markdown prose. Verify the
> examples against the rules by hand — in particular check Unicode code point
> ordering claims, casefold behavior, and whether each stated expected output
> actually follows from the rules as written. You may run python3 to check code
> points, casefold results and sort order, and to implement the rules and
> compare; do not edit the spec.
>
> Report only issues that would actually cause two competent implementers to
> write different programs, or that are factually wrong. For each, give
> numbered findings naming the rule or example, what is wrong, and a concrete
> fix. Do not pad the list with stylistic preferences, speculative edge cases
> that cannot arise in English, Russian or Kazakh news text, or suggestions to
> document things the rules already determine. If the spec has no significant
> issues, say exactly that and explain what you verified to reach that
> conclusion.
>
> Return ONLY the review markdown as your final message — it will be pasted
> into a file verbatim. Do not add a preamble, and do not summarize what you
> did.

---

# Review: `am_adilov/spec.md` — Top words

**The spec has no significant issues.** I could not find an error, a contradiction between a rule and an example, or an ambiguity that would lead two competent implementers to different programs. Below is what I checked and how.

## Reference-implementation check

I implemented the rules literally in the order rule 1 dictates (NFC over the whole text → mark deletion per rule 4 → word scan per rules 5–12 → per word: `casefold`, NFC, mark deletion, `ё`→`е`, joiner trim, drop-if-empty → `sorted(items, key=lambda kv: (-kv[1], kv[0]))[:10]`) on CPython 3.13.9 (`unicodedata.unidata_version == '15.1.0'`, as the interface section claims).

I then parsed the example table straight out of the file — rather than retyping the inputs — and `ast.literal_eval`'d both columns, so the characters actually in the file were the ones tested. **All 38 rows reproduce exactly**, including the `MyStr` subclass row, the `42` → `TypeError` row, and the lone-surrogate row. Dumping the code points of every non-ASCII-alphanumeric character in each input confirmed the literals are the characters the prose names: U+0027 vs U+2019 in the `don't`/`don’t` row, U+02BC in the `qaʼla` row, U+02BB in the `qaʻla` row, U+0149 and U+02BC in the `ŉ ʼn` row, U+0301/U+0335/U+0306 in the mark rows, U+0130 in the `İnternet` row, U+04AF/U+0443 in the `үй уй` row, U+2011, U+00AD, U+1F642, U+D800. No row uses a character the prose misidentifies.

## Ordering and casefold claims

Every code-point ordering claim holds:

- Rule 16, "U+002D being below `ы`": 0x2D < 0x44B, and `'кот' < 'кот-а' < 'коты'` — so the `"кот коты кот-а"` row's `[("кот",1),("кот-а",1),("коты",1)]` is right, and the prefix claim is right.
- Row `"үй уй"`: `у` is U+0443, `ү` is U+04AF, so `"уй"` precedes `"үй"`. Correct as printed.
- The unstated orderings the other rows depend on also hold: `"don't"` (U+0027 = 39) before `"don’t"` (U+2019 = 8217); `"qala"` before both `"qaʼla"` and `"qaʻla"` (at index 2, `l` = 108 < U+02BB/U+02BC = 699/700); `"do"` before `"re"`; `"hyphen"` before `"soft"`; `a`…`j` kept and `k` dropped in the eleven-way tie.

Casefold claims:

- `'ß'.casefold() == 'ss'`; `'İ'.casefold() == 'i̇'`; `'ŉ'.casefold() == 'ʼ' + 'n'`; `'I'.casefold() == 'i'` and `'ı'.casefold() == 'ı'`, so rule 13's dotless-`I` limitation is stated accurately; `'Ё'.casefold() == 'ё'`, which is what makes the `"ЁЖ ёж"` row work.
- `unicodedata.normalize('NFC', 'i̇')` leaves the mark in place (there is no precomposed `i`-with-dot-above), so rule 4's second pass is what deletes it and `İnternet` really does land on `internet`. This is load-bearing and correct.
- I searched all of Unicode: **U+0149 is the only character whose casefold emits any of the six joiners**, and **no character with `isalpha()` true casefolds to a string beginning with an `Mn`/`Mc` mark**. So rules 13/14 cover the complete set of shape-breaking casefolds, and applying the second mark-deletion pass per word (rule 1) can never differ from applying it to the whole text.

## Rule-by-rule factual claims

- Rule 5, "`ch.isalpha()` is true — exactly Unicode general category `L*`": verified exhaustively over all 0x110000 code points, zero mismatches.
- Rule 5's aside that the derived Alphabetic property is wider but not for modern Latin/Cyrillic/Kazakh: the extra Cyrillic Other_Alphabetic characters are the U+2DE0–U+2DFF Old Church Slavonic combining letters, which are outside modern orthography. Accurate. And `re` indeed has no `\p{Alphabetic}`.
- Rule 6: U+02BB/02BC/02BD are all category `Lm` and all return `isalpha() == True`, so the rule-5 exclusion is genuinely necessary, exactly as rule 6 explains.
- Rule 3, "the two letters Russian and Kazakh share that have a decomposed spelling": over the union of the Russian and Kazakh Cyrillic alphabets in both cases, exactly four code points have a canonical decomposition — U+0401/U+0451 (`Ё`/`ё`) and U+0419/U+0439 (`Й`/`й`). That is the two letters claimed, no more.
- Rule 3, the nine Kazakh-specific letters `ә ғ қ ң ө ұ ү һ і` (and their capitals): all nine have an empty `unicodedata.decomposition` and are NFC- and NFD-stable. Correct, including `ұ` (U+04B1) and `і` (U+0456), which are the two most likely to have been wrong.
- I also confirmed no Latin or Cyrillic letter is destabilized by NFC (`normalize('NFC', ch) != ch` has no Latin or Cyrillic hits), so the first NFC pass can never decompose a letter and let rule 4 silently strip its diacritic. The whole Kazakh Latin 2021 alphabet is NFC-stable, with `İ` the only letter whose casefold introduces a mark.
- Rule 12: U+2010 and U+2011 are `Pd` and U+00AD is `Cf` — none alphabetic, none `Mn`/`Mc`, so all three are separators as stated.
- Interface: `unicodedata.normalize`, `unicodedata.category` and `str.casefold` all accept a lone surrogate without raising (`'\ud800'` → category `Cs`, `isalpha()` false), so the "no other exception is raised" guarantee is actually satisfiable by a rule-following implementation. Separately, the final word always passes through `str.casefold()`, which returns a plain `str`, so the "plain `str`" guarantee holds even for a subclass input — note that `unicodedata.normalize` does *not* copy a subclass, but that cannot leak through.

## Differential test for latent ambiguity

The places where the spec's wording leaves any slack are the per-word vs. whole-text application of casefold/NFC/mark-deletion, the relative order of the `ё` fold and the joiner trim inside rule 14, and rule 1's loose phrase "combining marks" against rule 4's precise `Mn`/`Mc`. I built three alternative implementations embodying those alternative readings and fuzzed all four against each other on 60,000 random strings drawn from the full in-scope repertoire — English, Russian, Kazakh Cyrillic, Kazakh Latin, all six joiners, U+2010/U+2011/U+00AD, U+0301/U+0306/U+0308/U+0335, ZWJ, an emoji, digits, and Western and Russian punctuation. **Zero divergences in all three comparisons.** The slack is provably immaterial for the three languages in scope: the precise clauses in rules 4, 5 and 14 carry the weight, and rule 1's step list resolves the one place where order genuinely matters (deleting marks before NFC would turn `и` + U+0306 into `и` rather than `й`, which the `"й й"` row pins).

The only input I found on which a plausible re-reading could diverge is a word whose casefolded form contains a doubled joiner, which requires U+0149 `ŉ` adjacent to a joiner (`"ŉ-ŉ"` yields `n-ʼn` under rule 14's trim-only reading, versus two words under a re-split reading). U+0149 is deprecated and appears in no English, Russian or Kazakh orthography, so this is exactly the kind of case the brief excludes, and rule 1's step list resolves it in any event.

## Minor, non-divergent

The "Decisions pinned" column occasionally under-credits: the `"İnternet internet"` row cites only rule 13 although rule 4 does the mark deletion; the `"qaʻla ʻqala"`, `"soft­hyphen"` and `"re-'do re do"` rows omit rule 16 although their output order depends on it. Nothing an implementer could get wrong from this — the rules themselves are complete — so I am not listing these as findings.

---

## How the spec was checked, independently of the review

Each round's example table was also verified mechanically on my side: a script
parses the table out of `spec.md`, evaluates each input cell as a Python
literal, runs it through a reference implementation of the rules, and compares
with the expected output cell. All 38 rows match on CPython 3.13, and the
`TypeError` row is checked separately. This is what caught, in an earlier
round, that the invisible characters the examples depend on (U+00AD, U+0301,
U+0306, U+0335, U+2011) had been written into the table as literal characters
rather than `\u` escapes.

## Round history

| Round | Findings | What it caught | What I rejected |
| --- | --- | --- | --- |
| 1 | 18 | NFC rationale factually wrong (the Kazakh letters I named have no decomposition); `Unicode Alphabetic` unreachable from the stdlib; multiple joiners per word ambiguous; no numbered rule for the cut to ten | nothing substantive |
| 2 | 11 | "longest match at each position" permits overlapping words (`"cat"` → `at`, `cat`, `t`); Russian stress marks shredding words, which I had dismissed as malformed input; the returned spelling contradicted between two rules | U+02B9 as a joiner — it is MODIFIER LETTER PRIME, a palatalization mark, not an apostrophe; and duplicating rendered characters beside the `\u` escapes, since two renderings of one cell invite drift |
| 3 | 3 | a stale cross-reference in the interface; "a joiner sorts below any letter", false for U+02BB–U+02BD, which sit above `z`; rule 4 silent on single-pass versus iterative mark deletion | nothing |
| 4 | 0 significant | four editorial inaccuracies, including a self-defeating rationale: I had cited Kazakh Latin `I ı` as motivation for casefolding, but `"I".casefold()` is `"i"`, not `"ı"` | nothing; all four applied |
| 5 | 0 | no findings — the review above | nothing to reject |
