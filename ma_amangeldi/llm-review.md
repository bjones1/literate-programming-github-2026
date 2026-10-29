# LLM review of `spec.md`

Prompting **Claude Opus 5.5** (`claude-opus-5-5`, in Claude Code), **extra high effort**, with the prompt below produced the following output. It is the last review the LLM performed, and it reviewed the `spec.md` in this commit. Strikethrough indicates LLM output that I deemed incorrect or irrelevant. I did not change `spec.md` after this review; the optional suggestions only add examples or adjust wording.

## Prompt

```text
Review ma_amangeldi/spec.md, a specification for the Top words exercise (https://github.com/bjones1/literate-programming-book/blob/main/exercises/top-words.md).
```

## Output

### Review of `ma_amangeldi/spec.md` (Top words)

I found no real defects. The spec is ready, and every example is correct on Python 3.12, 3.13 and 3.14.

~~I reviewed the working-tree version, which has uncommitted edits compared with HEAD: the 3.14 claim, `str.__str__`, the ℹ case, extra calls in E12 and E13, and the Kazakh Latin note.~~

#### How I checked it

- **Two implementations:** I wrote two separate implementations of the spec. One is a hand-written scanner. The other is a regex built from explicit `L*` and `M*` character ranges. On 20,000 random strings of awkward characters they gave identical results.
- **Examples read from the file:** A script took every input and expected output straight from the table in `spec.md`, rather than from my retyping. That is all 20 rows, 37 separate calls, including the three `TypeError` cases. All pass with both implementations on Python 3.12.2 (Unicode 15.0), 3.13.5 (15.1) and 3.14.6 (16.0). The 3.12/3.13/3.14 claim on line 7 holds.
- **Unicode versions:** I compared every character that exists in Unicode 15.0 across the three Pythons: its class (letter, mark, joiner, separator), its normalized form and its combining class. Nothing changed. Only characters added after 15.0 behave differently (622 of them become letters or marks on 3.13, 4,975 on 3.14). So line 7's sentence about newer Pythons is accurate and complete.
- **Normalization side effects:** I searched all code points for normalization that changes a character's class. The only case is U+0345, which folds to `ι`. No letter normalizes into a separator, and no separator combines with a mark to make a letter. So normalizing the whole text before splitting causes no surprises.
- **Script mix-ups:** I listed every non-ASCII character in the file. All the Cyrillic and Kazakh text really is Cyrillic, and the Latin `a` in E5 and E16 really is Latin.
- **Edge cases:** Each of these gave one answer that follows directly from the rules:
  - `str` subclasses that override `__str__`, `casefold`, `replace`, `__iter__` and `__len__` count normally (`[('cat', 2)]`).
  - An object that fakes `__class__ = str` gets `TypeError` from `str.__str__`, which is also what the spec requires.
  - Lone surrogates, the BOM, ZWJ and ZWSP act as separators.
  - NBSP, tab and newline split words, so `inter-\nnational` becomes two words.
  - `ŉ` becomes `ʼn`, final sigma folds, `ǅ`/`Ǉ` fold, and `ẞ`/`ß`/`ss` match.
  - `a-'b a'b'` gives `a`, `a'b` and `b`.

#### Real defects

None.

- **Wrong examples:** none, including the code-point tie order in E5, E7, E10, E13, E16 and E17.
- **Contradictions:** none. Rules 1–8 match the pattern.
- **Inputs where two careful implementations disagree:** none found.

#### Optional suggestions

1. **The str-subclass decision has no example.** The exercise asks for an input and output for every decision. Python literals can't express a subclass, so add a short note under the table. For example, `class S(str): __str__ = lambda self: "x"` and `top_words(S("Cat cat"))` gives `[("cat", 2)]`.
2. **Show that a stress mark makes a different word.** `"по́ле поле"` gives `[("по́ле", 1), ("поле", 1)]`. E15 shows that marks stay, but not that they keep words apart. A Russian reader might expect the two to match.
3. **Show U+2019 used as a closing quote.** `"‘cat’ don’t"` gives `[("cat", 1), ("don't", 1)]`. Curly quotes are common in English news, and this shows the same character being both an apostrophe and a quote.
4. **Show NBSP and newline as separators.** For example, `"в Алматы\nв"` gives `[("в", 2), ("алматы", 1)]`. NBSP is common in Russian and Kazakh typography. No current example shows it.
5. ~~**Wording:** "keeps no state between calls" could be read as forbidding a module-level compiled regex. Something like "the result depends only on `text`" says what is meant.~~

#### Verdict

**Ready.** The spec is unambiguous and all 37 example calls are correct on Python 3.12, 3.13 and 3.14. The suggestions above add examples or improve wording; none is a fix.

~~#### Note on `llm-review.md`~~

~~`ma_amangeldi/llm-review.md` no longer describes the current spec. It records a review of the committed version: it mentions only Python 3.12 and 3.13, says only suggestion 1 was applied, and doesn't cover the new E10, E12 and E13 calls. If this review is its replacement or an added round, update that file.~~
