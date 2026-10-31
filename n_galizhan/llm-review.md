Prompting Claude Opus 5.5 (`claude-opus-5-5`), run as a Claude Code subagent with file and shell tools at its default effort, with the prompt below produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant. Notes marked **Response** are mine and say what I changed in `spec.md` afterwards.

This was the fifteenth review. Each review ran in a fresh context with the same prompt, except for the scratch directory, and I fixed each review's findings before running the next. Row and line numbers in the output refer to the version the review read. As a final check, I gave only `spec.md` to Claude Sonnet 5.5 and asked it to write `top_words.py`; its code produces every expected output in the table on Python 3.11 and 3.14.

**Prompt:**

```text
Review spec.md

The file is /Users/fine/literate-programming-github-2026/n_galizhan/spec.md (review only this file). It is a student's specification for this exercise from a course on writing specifications:

> A news site shows a "Trending words" box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh. Python 3. Name the function `top_words` and put it in `top_words.py`.
>
> Tighten this specification to produce one plausible result. Then, for each decision your spec makes, describe at least one input and its expected output, usually as a table in your spec. A useful example gets a different output under some other plausible reading of the original spec.

Find anything that would let two careful implementers build different programs: ambiguities, contradictions, factual errors, examples whose expected output does not follow from the rules, and missing decisions that matter for English, Russian or Kazakh news text. Also flag text that is redundant or longer than it needs to be. Rank findings by severity and be concrete (quote the line, give an input that exposes the problem). Do not rewrite the spec.

Do not modify any files in the repository. If you write code to check the examples, put it in /private/tmp/claude-501/-Users-fine-literate-programming-github-2026/03ab780c-66c2-44ec-8e99-5c5c439622f7/scratchpad/review15/ and run Python with `python3.14 -I` (3.11 is also installed as `python3.11`). Your final message is the review itself, in Markdown, addressed to the spec's author.
```

**Output:**

---

# Review of `n_galizhan/spec.md` (Top words)

**Bottom line:** I couldn't find a place where two careful implementers would build different programs. The algorithm is precise, and all 35 example rows check out. The real problems are elsewhere. The suffix rule in step 4 doesn't match its own stated rationale for Kazakh written in the 2021 Latin alphabet. The prose explaining that rule claims a narrower effect than the rule actually has. Some passages say the same thing twice.

**How I checked:** I implemented the Steps literally (`impl.py`). Every row in the Examples table matches under Python 3.11 (Unicode 14.0) and 3.14 (Unicode 16.0), including the `S("Cat")`, mutation, empty-input and `TypeError` rows. Your factual claims are also correct:
- U+2139 is `Ll`, U+02BB and U+02BC are `Lm`, U+200B is `Cf`.
- `"İ".casefold()` adds U+0307.
- NFC turns `е`+U+0300 into U+0450.
- NFKC turns `№` into `No`.
- `[^\W\d_]` matches `²` and `½`.

Unless noted otherwise, every expected output below comes from running that implementation.

---

## Findings, by severity

### 1. Medium-high: Step 4 removes Kazakh case endings in Cyrillic script but keeps most of them in Latin script

> An *ending* is a maximal run of letters that is all Cyrillic (U+0400 to U+052F) or at most two letters long.
> Such runs (`20th`, `1990-х`, `10-нан`, `«ҚТЖ»-ның`) are fragments, not words

The intro names the 2021 Latin alphabet as a supported input. In that alphabet, a suffix of three or more letters is not an "ending", so it is counted as a word. The same suffix in Cyrillic is dropped.

| Input | Output under the spec |
| --- | --- |
| `"мен «ҚТЖ»-мен"` | `[("мен", 1), ("қтж", 1)]` |
| `"men «QTJ»-men"` | `[("men", 2), ("qtj", 1)]` |
| `"2024-жылы 2024-jyly"` | `[("jyly", 1)]` |
| `"10-nan «QTJ»-nyñ 5%-ğa 1-şı"` | `nan` and `nyñ` are counted; `ğa` and `şı` are dropped |

The second row is exactly the merge that row 86 exists to rule out ("counting the `мен` of `«ҚТЖ»-мен`"). Within Latin Kazakh the rule also contradicts itself: two-letter endings go, longer ones stay.

You need to decide one of two things:
- Latin Kazakh endings are counted. Then say so next to the stated cost, and add a row.
- They are fragments like their Cyrillic forms. Then the ending test needs a different criterion than script.

> **Response:** Addressed with the first option. Step 4 now says that a Latin run of three or more letters is never an ending, so Kazakh written in Latin script keeps such suffixes as words, and a new row, `"men «QTJ»-men"` → `[("men", 2), ("qtj", 1)]`, pins it. I accept this cost: Kazakh news sites still publish mainly in Cyrillic, and treating Latin-script Kazakh differently from English would need the language detection that the spec rules out.

### 2. Medium-low: The "cost" sentence and row 87 say the rule drops less than it does

> The cost is that Cyrillic words joined to a number go too (`1-орын`, `30-летний`), as do short Latin words after such a `-`

Row 87 says it "Rules out: keeping a short word after `.` or `:` and `-`".

But the rule drops every Cyrillic word, of any length, after a hyphen that follows any non-letter: `:`, `.`, `»`, `)`, `%` or an emoji, not only a digit.

- `"Он ответил:-Хорошо."` gives `[("он", 1), ("ответил", 1)]`. The word `хорошо` is gone.

Someone reading only the prose would predict `хорошо` stays. The rule itself is unambiguous, so implementers won't split here. But the explanation misleads anyone checking your intent. Either make the sentence accurate or delete it (see R2).

> **Response:** Addressed. The inaccurate cost sentence is gone (see R2), and row 87's "Rules out" now names `eu` and `да` instead of "a short word".

### 3. Low: Your homoglyph decision (row 67) lets common misspelled suffixes through step 4

Row 67 keeps Latin `i` inside Cyrillic words, which is a defensible choice. Kazakh web text often types Latin `i` for `і`. A suffix containing it is no longer "all Cyrillic", and at three or more letters it becomes a word:

- `"«ҚТЖ»-нiң"` (Latin `i`) gives `[("нiң", 1), ("қтж", 1)]`.
- `"1-шiсi"` gives `[("шiсi", 1)]`.

If this is accepted, say so in one line.

> **Response:** Accepted without a change. A run that mixes Latin and Cyrillic letters is not all Cyrillic, so it follows the two-letter limit; the spec does not try to repair misspellings.

### 4. Low: Step 4 looks only one character back, so near-identical inputs behave differently

None of these is ambiguous, but the stated rationale doesn't cover any of them, and no example pins them down:

| Input | Output | Compare with |
| --- | --- | --- |
| `2-ші-орын` | `[("орын", 1)]` | `2-орын` gives `[]` |
| `5--ге` | `[("ге", 1)]` | `5-ге` gives `[]` |
| `XXI-ші` | `[("xxi-ші", 1)]` | `21-ші` gives `[]` (Roman numerals are letters) |
| `25°C` | `c` is counted | `°` is not `Nd` |

The degree sign also has a Russian twist: `+25°С с ветром` gives `[("с", 2), ...]`, because the Cyrillic `С` after `°` merges with the preposition `с`.

If you count any of these as fragments, the rule needs another clause. If not, nothing needs to change.

> **Response:** No change, as the review allows. Each case follows the one-character look-back rule, and I do not count them as fragments.

### 5. Low: Row 69 locks in uneven handling of stress marks

Row 69 rules out "keeping the acute mark", so the intent is that stress marks don't create new words. Yet:
- `всѐ` becomes `всѐ`, a separate word, because a precomposed U+0450 exists.
- `все́` becomes `все`.
- Likewise `ѝ` becomes `ѝ`, but `и́` becomes `и`.

Whether a stress mark survives depends on whether Unicode happens to have a precomposed character for it. If that is acceptable, a short note saying it is a known side effect would help. Right now the example presents it as a goal ("deleting a grave accent that NFC composes into `ѐ`" is listed as a wrong reading).

> **Response:** Kept on purpose. Deleting only the marks that NFC cannot compose is what keeps `й` and `ё` intact; the grave-accent example records the side effect so that no one "fixes" it.

### 6. Low: The last condition sentence in step 4 is hard to read

> …right after a `-` whose preceding character is not a letter, `-`, U+200B or whitespace (`str.isspace`), or right after a `'` whose preceding character is a digit.

There are two different "or"s. The second branch (`'` after a digit) reads at first like a fifth item in the "not" list. Splitting the two triggers into two sub-bullets would fix this without changing any meaning.

> **Response:** Addressed. Step 4 now lists the two triggers as separate bullets.

---

## Redundant or longer than needed

- ~~**R1. Step 1's last two sentences** ("Casing is not Turkic… `BILIM` gives `bilim`, not `bılım`") repeat row 71. Keep one of them. The row is more precise.~~

  > **Response:** Not changed: the sentence gives the reason for the decision (English `In`), which the row cannot show.

- **R2. Step 4's last three sentences** ("Such runs… so this is not stemming; endings joined to letters… stay. The cost is…") are rationale. They repeat rows 79–87, and the cost part is incomplete (finding 2). Cutting them loses nothing that the table doesn't already state.

  > **Response:** Addressed; these sentences are replaced by one accurate clause about endings after letters and Latin runs.

- ~~**R3. Too many rows for step 4.** Nine of the 35 rows (79–87) test that one step, and several overlap:~~
  - ~~Rows 80 and 86 both test `«ҚТЖ»-` plus a suffix. Row 86 alone covers it if you add `(ҚТЖ)-ға` to it.~~
  - ~~Rows 79 and 85 both test endings after a number (`20th`, `1990-х`, `1-ші` / `20th-century`, `1-орын`, `10-нан`).~~
  - ~~Rows 82 and 83 both test a `-ке` that is kept: at the start of the text, and after U+200B.~~

  ~~About five rows would rule out every reading these nine do.~~

  > **Response:** Not changed: each of these rows pins a reading that an earlier review found missing (start of text, U+200B, the two passes of Step 4), and merging them would hide which reading each one rules out.

- ~~**R4. Row 88 (`ℹ️cat`)** documents a strange result (`ℹcat`) for input that almost never appears in articles. The letter definition is already shown by `ʻ` in row 74. It is a candidate for deletion.~~

  > **Response:** Not changed: it is the only row showing that a symbol-like letter (`ℹ`, category `Ll`) stays in a word, which an earlier review asked for.

Rows 91 and 92 (stop words; stemming and transliteration) are worth keeping. For a "Trending words" box, stop-word removal is a very plausible reading.

---

Scripts used for checking are in `/private/tmp/claude-501/-Users-fine-literate-programming-github-2026/03ab780c-66c2-44ec-8e99-5c5c439622f7/scratchpad/review15/`:
- `impl.py`: the literal implementation
- `check.py`: every example row
- `probe.py`: the inputs above

No repository files were modified.
