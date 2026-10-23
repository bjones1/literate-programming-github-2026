Prompting Claude Opus 5.5 with `Review spec.md` produced the following output;
strikethrough indicates LLM output that I deemed incorrect or irrelevant.

---

I'll check the spec by building a reference implementation and running every
example against it.

I wrote a straightforward implementation from the rules alone and ran all 52
examples against it. All 52 pass, and the rule-1 subclass cases pass too. The
rules don't contradict each other or the examples, and the ordering claims in
rule 9 are correct (all Kazakh-specific letters fall in U+0456–U+04E9, after
Russian `а`–`я`/`ё`). The issues are about accuracy and missing tests.

## 1. The "known limitation" in 2.2 is understated (real issue)

In the 2021 Kazakh Latin alphabet, `I` is the capital of `ı` and `İ` is the
capital of `i`. So `str.lower()` gets every capitalized `ı`-word wrong, not only
all-caps ones. That includes sentence-initial words and title-case headlines,
which are common in news. `Ilım` becomes `ilım`, not `ılım`.

Example 43 shows this: `'Ilim'` is title case, not all-caps, yet the note says
only all-caps words are affected.

Fix: reword it as "any word written with capital `I` where `ı` is meant
(sentence-initial, title case or all caps) is miscounted." Add an example like
`'Ilım ılım'` → `[('ilım', 1), ('ılım', 1)]`.

## 2. Rule 1 doesn't test the most likely trap: `__str__`

The obvious conversion, `str(text)`, calls a subclass's `__str__` override, and
none of the examples catch that. Add one:

`class U(str): def __str__(self): return 'dog'` → `top_words(U('cat cat'))`
returns `[('cat', 2)]`.

(`str.__str__(text)` gives a plain copy without using overrides.)

## 3. Minor coverage gaps

- U+200F is listed in 2.1 but never tested. Example 44 covers U+200E, U+200C,
  U+200D, U+2060 and U+FEFF, and example 35 covers U+200B.
- Examples 3, 4 and 26 are labelled rule 3, but they mostly exercise lowercasing
  and counting. Nothing tests rule 3 with a non-Latin, non-Cyrillic letter such
  as `ª` (category Lo).

## 4. Design note, not a bug

Deleting U+200B joins the words on either side of it (example 35). That's fine
if editors only use it inside words. But some CMS and copy-paste pipelines put a
zero-width space between words with no regular space. If your article source
does that, words will merge silently. Treating U+200B as a separator is the
safer choice, unless example 35 reflects real data.

Rules 2–12 need no other changes.