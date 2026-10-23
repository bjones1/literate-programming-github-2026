# LLM review of `spec.md`

Prompting Claude Sonnet 5.5 (default effort; no extended-thinking setting was chosen) with `Review spec.md` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

The reviewed text is the shortened `spec.md` in this commit (66 lines).

---

## Output

### Overall

The spec is short, deterministic, and testable. The seven-step algorithm fixes every normalization choice that could change a result (NFC vs NFKC, `lower()` vs `casefold()`, what is deleted, what joins words). The table gives an input and expected output for each decision. I found no contradictions between the algorithm and the table, and no row whose expected output follows from a different reading than the one the algorithm states.

### Findings

| # | Severity | Finding | Suggestion |
|---|---|---|---|
| 1 | Low | `isalpha()`, the `M*` category test, and NFC all depend on the Unicode database bundled with the Python runtime. For rare characters, results can differ between Python 3.9 and later releases. | Accept this explicitly, or pin a Python version. |
| 2 | Low | Digits next to letters leave fragments: `1st` becomes `st`, `COVID-19` becomes `covid`, `H2O` becomes `h` and `o`. This follows from "digits separate", but the table only shows `abc123def`. For a "Trending words" box, `st` could rank surprisingly high. | Add a row such as `1st 2nd` giving `nd: 1, st: 1`, or state in prose that fragments are intended. |
| 3 | Low | The Cyrillic exception (`г`/`к` + U+0301 composing to `ѓ`/`ќ` under NFC) is documented, but a stressed `г` or `к` then counts as a different word from the unstressed one. Deleting U+0301 before NFC would avoid this, at the cost of dropping the accent on decomposed Latin `é`. | Keep as is if the trade-off is deliberate; it is stated clearly enough to implement. |
| 4 | Low | No size or performance expectation is given. | Irrelevant for article-length text; mention only if very large inputs are expected. |

### Verdict

Ready to implement. None of the findings changes any expected output in the table. The next useful step is to generate `top_words.py` from the spec and run every table row as a test.
