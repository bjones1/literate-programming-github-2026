# LLM review of spec.md

Prompting Claude Opus 5.5 (`claude-opus-5-5`), default effort, with the prompt
below produced the following output; strikethrough indicates LLM output that I
deemed incorrect or irrelevant. This is the last review performed on the
current `spec.md` (the sixth round; I fixed the findings of the earlier rounds
before running it). The only edit to the output is that two absolute local file
paths are replaced by `[local path omitted]`.

## Prompt

```
Review anu_kurmanov/spec.md (in the current directory). It is a specification for this problem:

"A news site shows a "Trending words" box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh. Implementation: Python 3. Name the function `top_words` and put it in `top_words.py`. Assignment: Tighten this specification to produce one plausible result. Then, for each decision your spec makes, describe at least one input and its expected output, usually as a table in your spec. A useful example gets a different output under some other plausible reading of the original spec."

Find errors (including wrong expected outputs in the tables), contradictions, and underspecified behavior that would let two careful implementers produce different results. Do not edit any files. Report a numbered list of findings, each with the location, the problem, and a suggested fix; put the most serious first. If something is fine, don't list it.
```

## Output

I reviewed [local path omitted]/anu_kurmanov/spec.md and edited nothing in the
project. I found no wrong expected outputs and nothing that would let two
careful implementers get different results. What's left is four small wording
problems.

**How I checked:** I wrote a reference implementation that follows the spec
literally:

- type check, then `str.__str__`, then NFC on the whole text;
- scan for words using the letter, mark and joiner rules;
- for each word: replace ’ with ' and ‐ with -, apply `lower()`, then NFC again;
- sort by count (highest first), then by `str` order, and keep 10.

It's in the scratchpad at [local path omitted]/ref.py.

I ran every table row through it on Python 3.14 (Unicode 16.0). Every row
matches, and every `raises TypeError` row raises. The prose claims hold too:

- The `Shout` and `W` subclass examples return `[("hi", 2)]` as a plain `str`.
- Lowering `ΑΣ.Β` as one text gives `σ`; lowering just the word gives `ς`.
- `ΑΣ'Β` lowers to `ασ'β`, while `ΑΣ-Β` and `ΑΣ‐Β` both lower to `ας-β` /
  `ας‐β`. So the order of the ’/‐ replacement and `lower()` cannot change any
  result.
- A lone surrogate is simply a separator; nothing crashes.

**Findings, most serious first (all minor):**

1. **Section 2, line 78 contradicts section 4, line 173.** Line 78 says "Apart
   from lowercasing (section 3), no other folding happens". But section 4 also
   folds ’ (U+2019) into ' and ‐ (U+2010) into -~~, and NFC is itself a kind of
   folding~~.
   - Fix: "Apart from NFC, lowercasing (section 3) and replacing the joiners ’
     and ‐ with ' and - (section 4), no other folding happens."

2. **Sections 3 and 4: the steps applied to each word are never listed in one
   place.** Lowering and the second NFC are in section 3; the joiner
   replacement is in section 4 ("After a word is found..."). Nothing says which
   comes first. Today the order makes no difference, but a reader has to work
   that out using case-ignorable properties. ~~A future change, such as adding
   another joiner, could quietly make the order matter.~~
   - Fix: add a numbered list at the top of section 4 or in section 1: (1) type
     check and `str.__str__`, (2) NFC on the whole text, (3) scan for words,
     (4) for each word: replace joiners, `str.lower()`, NFC, (5) count, sort,
     keep 10.

3. **Section 1, line 29: wrong cross-reference.** "(the same Unicode database,
   see the next bullet)" points to the Unicode-database bullet, but that bullet
   comes before this one (lines 20–24), not after.
   - Fix: change "see the next bullet" to "see above" or "see the second
     bullet".

4. **Section 8, lines 389–392: a requirement is listed under "Out of scope".**
   "The only requirement is O(n log n) time or better" is a requirement placed
   in a section about what the spec does not cover. ~~It also has no example,
   unlike every other decision.~~
   - Fix: move the complexity bound to section 1 as a non-functional
     requirement and keep only "performance tuning" in section 8. Or drop the
     bound if it isn't meant to be checked.
