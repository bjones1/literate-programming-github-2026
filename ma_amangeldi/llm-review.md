# LLM review of `spec.md`

Prompting **Claude Opus 5.5** (`claude-opus-5-5`, in Claude Code) with the prompt below produced the following output. This is the last of two reviews. The first review found three defects (unpinned Unicode version, the "discarded" stray-mark rule, and `str` subclasses), which I fixed before this review. Strikethrough marks output I judged irrelevant. After this review I applied suggestion 1.

## Prompt

```text
Review spec.md. It is a specification for this exercise: "A news site shows a
'Trending words' box beside each article. Write a Python function that finds
the 10 most common words in an article's text, along with how many times each
one appears. Articles may be in English, Russian, or Kazakh. Name the function
top_words in top_words.py. Tighten this specification to produce one plausible
result; for each decision give an input and expected output."

Find errors, contradictions, and underspecified behavior: any input on which
two careful independent Python implementations of this spec could disagree,
and any example whose expected output is wrong under the stated rules (you may
write and run your own implementation to check). Separate real defects from
optional suggestions. Do not edit spec.md. Output the review as markdown,
concise, ending with a verdict.
```

## Output

**How I checked it:** I wrote my own implementation from the spec and ran every example against it, E1–E20 including all multi-call cells and the three `TypeError` cases. It ran on Python 3.12.2 (Unicode 15.0.0) and 3.13.5 (Unicode 15.1.0). All examples pass on both, so the claim that 3.12 and 3.13 give the same results holds.

I also tried edge cases. Each gave one answer that follows directly from the rules:

- **Case folding creates a letter:** U+0345 folds to `ι`, so `"ͅx"` gives `ιx`. This is consistent because the spec classifies characters after normalization.
- **Ligatures:** `ﬁ` folds to `fi`.
- **Titlecase:** `ǅ` folds to `ǆ`.
- **Letter-numbers:** `Ⅻ` is category `Nl`, so it is a separator and gives no word.
- **Lone surrogates:** treated as separators.
- **Invisible and middle characters:** ZWJ and `·` split words.
- **Mixed joiners and trailing joiners:** `a-'b` gives `a` and `b`; `a'b'` gives `a'b`.
- **Other apostrophe-like letters:** `ʻ` (U+02BB, `Lm`) stays part of the word.
- **Turkish dotted capital:** `İstanbul` and `ISTANBUL` count as two different words, as E13 says.

### Real defects

None found.

- **Wrong examples:** none. Every expected output, including the code-point tie order in E5, E7, E16 and E17, is correct under the stated rules.
- **Contradictions:** none. The numbered rules 1–8 match the pattern, including "after a letter or mark", "joiner before a mark splits", and when U+2019 is replaced.
- **Inputs where two careful implementations disagree:** I could not find one. Normalization is a single exact expression, character classes are exact category tests, the pattern is deterministic, and sorting and cutting at 10 are fully defined.

### Optional suggestions (wording only)

1. **Say where scanning resumes after a word.** "At each position, the longest match … is a word" does not say that. Read literally, `at` and `t` inside `cat` would also be words. E1 rules this out, but one sentence would close the gap: "after a word, scanning continues at the first character after it."
2. **Say how to read a subclass without its overrides.** For example, call `str` methods directly, or note that `str(text)` may call an overridden `__str__`. This only helps the implementer; the required behaviour is already clear.
3. **Optionally add an example for case folding that creates a letter** (`"ͅ"` gives `[("ι", 1)]`). It would show that classification happens after normalization. The rule is already unambiguous.
4. ~~**Optionally say outright that the zero-width characters U+200C and U+200D are separators.** The rules already make this so. It matters little for English, Russian and Kazakh.~~

### Verdict

**Ready.** The spec is unambiguous and all 20 examples are correct on Python 3.12 and 3.13. The suggestions above are optional clarity edits, not fixes.
