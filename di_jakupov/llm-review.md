# LLM review of spec.md

I reviewed spec.md four times, each time in a new session with no memory of
the earlier rounds, and fixed the spec after each round. This file is the
output of the fourth (last) review.

LLM: Claude Opus 5.5, run as an agent in Claude Code, so it could write and run
its own implementation.

Prompt (only the local folder path is shortened to `<path>`):

```
Review spec.md. It is my specification for this exercise:

"A news site shows a 'Trending words' box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh. Python 3. Name the function top_words and put it in top_words.py. Tighten this specification to produce one plausible result. Then, for each decision your spec makes, describe at least one input and its expected output."

The spec is at: <path>/di_jakupov/spec.md

Find errors, contradictions, and underspecified behavior: places where two careful implementations could give different results, rules that behave badly on real English, Russian or Kazakh news text, and examples whose expected output is wrong. Write your own implementation from the rules alone (in a scratch directory, not next to spec.md) and check every example. Also say if the spec is longer than it needs to be. Do not edit spec.md. Report findings grouped as "Must fix" and "Minor", each with a concrete input and what goes wrong, plus a one-paragraph overall summary. Write it as a review I can paste into a markdown file.
```

Strikethrough marks output that I decided was incorrect or irrelevant. My
notes are the lines starting with "My note". Everything else I accepted and
fixed in spec.md after this review.

---

# Review of spec.md (top_words)

I wrote my own implementation from the rules alone and did not look at any
existing code. It is in [path omitted]/top_words.py, and the checks are in
check.py, probe.py, real.py and fuzz.py in the same folder. I ran it on Python
3.13.7 (Unicode 15.1). All 24 example rows (26 inputs, counting the two
TypeError cases) give exactly the expected output. I also wrote a second
version that tokenizes with a regex over character classes instead of a
hand-written scanner. The two agreed on 200,000 random strings built from the
tricky characters (marks, soft hyphen, ZWJ, ı/İ, the joiners, dashes, ß, е/ё).
I found no wrong examples, no contradictions, and nowhere that two careful
implementations of the text as written would disagree. The only exception is
the Unicode version difference, which the spec already admits. The problems
below are about how the rules behave on real news text, not about precision.

## Must fix

1. Function words fill the box. Rule 4 says there is no stop-word list. That
   makes the spec consistent, but the "Trending words" box stops being useful
   for English and Russian. Here is a four-sentence English budget story I
   wrote:
   `[('the', 11), ('said', 3), ('to', 3), ('and', 2), ('budget', 2), ('in', 2), ('is', 2), ('new', 2), ('of', 2), ('on', 2)]`
   Seven of the ten are function words. "parliament" and "schools" also have
   count 2, but they are cut because they sort after "on". The same story in
   Russian gives `('в', 6), ('на', 3), ('бюджет', 2), ('и', 2), ('что', 2), ...`.
   Kazakh does a bit better, but `екенін` and `тиіс` still get in.
   ~~The cheapest fix is a short fixed stop-word set per language, written out
   in the spec, plus one example row such as `"the cat and the dog"` ->
   `[('cat', 1), ('dog', 1)]`.~~ If you deliberately keep the literal "most
   common words" reading, say in one sentence that the box will mostly show
   function words and that this is accepted. "Short words count like any other
   word" describes the rule, not this consequence.

   My note: I keep the literal reading. The task asks for the most common
   words, and any stop-word list for three languages would be my own arbitrary
   choice and would make the spec much longer. I added the sentence to rule 4.

2. ~~Single letters from initials and abbreviations. Russian and Kazakh news
   write initials before surnames and abbreviate "year", so one-letter tokens
   rank high: `"Қ. Тоқаев 2024 ж. қыркүйекте Астанада сөйледі. Қ. Тоқаев 2025 ж. жоспарын айтты."`
   -> `[('ж', 2), ('тоқаев', 2), ('қ', 2), ...]`. `"The U.S. and the U.K. signed it."`
   -> `('u', 2)`, plus `('k', 1)` and `('s', 1)`. `"т.е."` -> `('т', 1)` and
   `('е', 1)`. The spec's own example shows this and calls it expected.~~

   ~~A one-line fix is "a word must contain at least two letters". It also
   removes "в", "и", "с" and "a". It changes the "Stop words, possessives,
   abbreviations", "Ties in code point order", "Cut after sorting", "Hyphen at
   the edge" and "Other dashes" examples, which use one-letter words, so those
   inputs would need longer words.~~

   My note: rejected. "в", "и", "с" and "a" are real words in Russian and
   English, and a minimum length is the same kind of filtering as a stop-word
   list. The spec already shows this behavior on purpose.

3. ~~The possessive splits the most important name in two.
   `"Tokayev Tokayev’s Tokayev's Tokayev"` -> `[('tokayev', 2), ("tokayev's", 2)]`.
   In English news the main person or country very often appears in the
   possessive, so the top entity shows up twice with half its count each. A
   fix: after tokenizing, remove a final `'s`, then a final `'`. This leaves
   "don't" alone and turns "it's" into "it". Replace the "Tokayev's" example
   with `"Tokayev Tokayev's"` -> `[('tokayev', 2)]`.~~

   My note: rejected. Removing `'s` is a small piece of stemming for English
   only, and the spec says there is no stemming. It also turns "it's" into
   "it", which is a different word.

## Minor

1. No stemming hurts Russian and Kazakh a lot.
   `"Қазақстан Қазақстанның Қазақстанда Қазақстанға"` gives four words with
   count 1 each. In Russian, "парламент" and "парламенте" are split, and so are
   "школы" and "школ". This is a documented decision and hard to fix with only
   the standard library, but say in one sentence that inflected forms count
   separately. Rule 4 says "no stemming", but it does not say what that means
   for a Kazakh article.

2. ~~In a short article the end of the box is alphabetical noise. Once the
   words with count 2 or more run out, the box fills with count-1 words in
   code point order. The Russian sample ends with `('администрации', 1),
   ('ближайшее', 1), ('будет', 1), ('время', 1), ('деньги', 1)`. Consider "only
   words that occur at least twice". Then `"cat dog cat"` -> `[('cat', 2)]`.~~

   My note: rejected. The task asks for the 10 most common words, and a word
   that occurs once is still one of them in a short article.

3. ~~The box shows forms that look wrong. Every word is shown casefolded:
   "tokayev", "astana", "қазақстан". For Kazakh Latin, the ı/i merge makes the
   box show `ilim` for `Ilım`/`ılım` (ілім). That is a misspelling in the 2021
   alphabet, and the same form is also used for `İlim` (илім):
   `"Ilım ılım İlim"` -> `[('ilim', 3)]`. The spec admits the counting cost of
   the merge but not the display cost. Either say it, or show the most
   frequent original spelling of each word.~~

   My note: incorrect. Rule 1 already says the returned words are in the
   normalized form and gives "ilim" for "Ilım" as the example. Keeping the
   original spelling would need a second rule for choosing between spellings.

4. The tie order is not alphabetical for Russian or Kazakh. Code point order
   puts ё and all Kazakh letters after я: `"ё я ә і а"` ->
   `[('а',1), ('я',1), ('ё',1), ('і',1), ('ә',1)]`. It is deterministic and the
   spec says so, so this is only cosmetic. A one-line note that this is not
   dictionary order would stop a reader from "fixing" it.

5. ZERO WIDTH SPACE (U+200B) is not deleted. CMS output sometimes puts it
   inside words: `"Kazakh\u200bstan Kazakhstan"` -> `kazakh`,
   `kazakhstan`, `stan`. Treating it as a separator is defensible (it is a
   space), but the spec deletes U+200C, U+200D and U+2060 and says nothing
   about U+200B. Add it to step 2, or state that it separates.

   My note: I kept it as a separator and said so in step 2, with an example.

6. ~~Other apostrophe look-alikes split words. `` "don´t don`t don′t" ``
   (U+00B4, U+0060, U+2032) -> `[('don', 3), ('t', 3)]`. People do type these
   in Russian and Kazakh text. Either add them to step 3 or say that only ’
   and ʼ are folded.~~

   My note: rejected. Step 3 lists exactly which characters are replaced, so
   the behavior is already defined. These three are an accent, a backtick and
   a prime, not apostrophes.

7. ~~URLs and e-mail addresses become words.
   `"https://tengrinews.kz/news info@example.com"` -> `https`, `kz`, `com`,
   `example`, `info`, `news`, `tengrinews`. "Markup already removed" does not
   remove URLs written in the text. Either accept this in one sentence or
   exclude them.~~

   My note: irrelevant. The input is the article text, and the behavior on
   URLs already follows from rule 3.

8. The Straße example is outside the three languages. For English, Russian
   and Kazakh, casefold() differs from lower() mainly on ligatures from copied
   PDFs. `"ﬁnance finance FINANCE"` -> `[('finance', 3)]`, while lower() would
   keep `ﬁnance` separate. That is an in-scope example of the same decision
   and could replace Straße in both the example table and the sentence under
   rule 1.

9. ~~Rare oddities you can ignore or mention in one line. Fullwidth letters
   stay separate (`"ＣＯＶＩＤ covid"` gives two words). A variation selector
   U+FE0F after a letter stays in the word because it is category Mn.
   "2024-жылы" is counted separately from "жылы".~~

   My note: irrelevant, as the review itself says.

## Length

The spec is about 930 words, and nearly all of it carries weight. A few things
could go:

- ~~"Each replacement in steps 2 to 4 is one str.replace() call". It only
  matters for inputs like `"i\u0307\u0307"`.~~ My note: kept. The third
  review found that exact ambiguity, and this sentence fixes it.
- "Return a new list". It cannot be tested and adds nothing.
- ~~The "Unicode hyphens join too" row could be merged into "Hyphen inside a
  word".~~ My note: kept as a separate row, because it shows a separate
  decision (step 3 of rule 1).
- Step 2 deletes U+200C, U+2060 and U+FEFF but has no example for them.
  Either add one input or accept that the soft hyphen and ZWJ rows cover the
  decision.

Trimming these saves about 10%. The spec is not much longer than it needs to
be. Adding a stop-word list and a minimum length would add more than these
cuts remove, and that is fine.

## Summary

As a precise specification this is strong. Every example is correct when
checked against an independent implementation. The tokenizing grammar is exact
enough that a scanner and a regex version agreed on 200,000 random tricky
strings. The normalization steps (NFC, casefold, deletions, ı/İ merge, NFC
again) are ordered correctly, and each has an example. The weak point is the
product, not the precision. On real news text the literal "10 most common
words" rule fills the box with "the", "в", "на" and single letters from
initials like "Қ." and "ж.", and English possessives split the main name in
two. These are deliberate, documented choices, so they are not errors. But for
something called "Trending words" they are the parts most worth changing or
explicitly justifying before you hand this in.
