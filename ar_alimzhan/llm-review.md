# LLM review of `spec.md`

Prompting Claude Opus 5.5, high effort, with `Review spec.md` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

This is the last review I ran. Earlier reviews found real gaps that I fixed in `spec.md` before this one: variation selectors (category `Mn`) were described as separators while the Mark rule would have kept them inside words; the joiner rule in section 4.2 did not repeat the "word in progress" condition from the table; the 1 MB limit did not say bytes or characters; test inputs did not say they are Python literals; `str` subclasses, locale-independent case folding (`İ`), lone surrogates and the 2018 apostrophe-based Kazakh Latin alphabet were not addressed; and test 14 was ambiguous. All 18 acceptance tests were checked against a reference implementation and pass.

---

## Review output

**Overall:** The specification is complete and internally consistent. The interface, input validation, processing order, word definition, counting key, ranking and truncation are all defined precisely enough that two independent implementations should return identical results for every input. I found no contradictions and no significant gaps.

**Checks performed**

- Processing order: NFC on the whole input → tokenize → replace `’` with `'` → `casefold()` → NFC on the key → count → sort → truncate. Unambiguous, and test 5 (decomposed `é`) and test 6 (`Straße`) pin down the two normalization points and the no-re-splitting rule.
- Character classes are defined by `unicodedata.category()` with explicit exceptions (joiners, variation selectors). Every code point falls into exactly one class.
- The joiner rule (word in progress **and** next character is a Letter) is consistent between the table and the scanning description, and tests 7, 8 and 9 exercise both the joining and the non-joining cases.
- Ranking (count descending, then code-point order) is total, so the output is deterministic; test 13 confirms independence from word order and test 14 confirms truncation with ties.
- All 18 expected results in section 7 follow from the rules as written; I re-derived each one and found no mismatches.

**Minor observations (not blocking)**

1. ~~The spec should define what happens when `text` is longer than available memory.~~
2. Results depend on the Unicode database of the running Python version (section 4 says so). A newly assigned letter could change a result between Python versions. This is stated and acceptable.
3. ~~Consider removing stopwords such as "the" and "и", because a "Trending words" box full of function words is not useful to readers.~~
4. Keys are returned in folded form (`strasse`, not `Straße`). This is stated in section 4.3; the UI may want to display a nicer form, but that is outside this function.
5. ~~Add stemming so that `Kazakhstan's` and `Kazakhstan` are counted together.~~
6. ~~The ellipsis character U+2026 is not mentioned.~~

**Conclusion:** No significant issues. The specification is ready for implementation.

---

**Why the struck-out items were rejected**

- (1) Memory limits are a property of the environment, not of this function; section 6 already states memory is proportional to input size.
- (3) and (5) contradict deliberate decisions in section 4.3 and 4.2: the task asks for the most common words, and stopword filtering or stemming would add language-specific rules for three languages that are not requested. The spec explicitly leaves stopword filtering to the caller and has a test (16) for the no-stemming decision.
- (6) U+2026 is punctuation (`Po`), so the existing "everything else is a separator" rule already covers it.
