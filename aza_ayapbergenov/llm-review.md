# LLM Review of spec.md (revised version)

Prompting Claude Sonnet 5.5 (thinking effort: medium effort) with the prompt below, plus the full text of the revised `spec.md`, produced the following output. Strikethrough indicates LLM output that I deemed incorrect or irrelevant.

This review was run after I revised `spec.md` in response to the instructor's review (Claude Opus 5.5, medium effort). It replaces my earlier review (ChatGPT GPT-6), which is no longer the last review performed.

## Model and settings

- Model: Claude Sonnet 5.5 (Anthropic), used in the Claude chat interface
- Thinking effort: medium effort
- Review date: October 24, 2026
- Tools: the model had a Python 3.12.3 sandbox (Unicode database 15.0.0) and was asked to run the examples

## Prompt used

Review the attached specification `spec.md` for a Python 3 function named `top_words`, which returns the 10 most frequent words in an article. The article may contain text in any language.

Identify ambiguities, contradictions, incorrect expected outputs, missing edge cases, and any mismatch between the rules and the examples.

Pay particular attention to:

- Unicode word boundaries, including combining marks and scripts other than Latin and Cyrillic
- Unicode normalization (NFC versus NFKC) and case-folding, including characters whose case-folding produces several code points or combining marks
- Punctuation, apostrophes, hyphens, underscores, digits, and number characters that are not digits
- Frequency counting and tie-breaking, including the exact comparison rule
- The top-10 limit and ties at the cutoff
- Fewer than 10 distinct words and empty input
- The exact return type and output ordering
- Invalid input types, including `str` subclasses and `bytes`
- Whether every example produces exactly its stated output. Write a reference implementation that follows only the text of the specification, run every row of Section 8 against it, and report each mismatch.

Do not silently choose behavior that the specification leaves undefined. List each problem, explain why it matters, and suggest a precise correction.

## Review output

I wrote a reference implementation from the spec text only and ran every Section 8 row through it. All 34 rows pass, and I found no incorrect expected output. The problems are in the spec's wording, scope and test coverage.

The run used Python 3.12.3 with Unicode 15.0.0. The table was parsed straight from the file, so there are no transcription errors. I also checked that my tokenizer gives identical output to regex.findall(r'(?:\p{L}\p{M}*)+') on 200,000 random strings, so the prose and the notation agree. The spec file has no Latin/Cyrillic homoglyphs.

Other things I verified:

The folded form of every code point is a fixed point, so a returned word fed back in returns itself.
No letter or mark folds into a separator, so folding never splits a word.
Only U+0345 turns a mark into a letter, so the spec's example covers the only such case.
Weaknesses in the examples

I broke the implementation in 18 ways that each violate one spec rule and checked whether the 34 rows catch them. Seven mutants survive, which means seven rules are stated but not tested.

#	Problem	Why it matters	Fix
1	Tie-break before the cut is untested. An implementation that takes most_common(10) and then sorts passes all rows.	The spec stresses this rule, but in the cut-off row the tied words appear in code-point order, so insertion order equals the right answer.	Add a row with the tied words in reverse order (row R3 below).
2	Normalization steps 1 and 3 are untested. Skipping either one passes every row.	Both change results. Step 3 matters for U+01F0 and U+0390, among 26 BMP characters I found. Step 1 matters for "\u0345\u20e5".	Add rows R6–R8.
3	Tie order is code-point, not UTF-16 or UTF-8 order, but no row proves it.	JavaScript, Java and C# compare UTF-16 units, so astral characters sort differently.	Add row R4.
4	Subclass handling has no example. Section 9 says subclasses are accepted, and a mutant that rejects them passes.	See the next section.	Add rows R1–R2.
5	Me (enclosing marks) and Lt are untested. Lt also cannot occur after case-folding: only Ll, Lo, Lm, Lu, Mn, Mc and Me survive.	The Lt listing is harmless but vacuous.	Add row R5, and a note that Lt never survives folding.
Defects in the spec text
The subclass rule is a real trap.
Section 9 says a subclass is processed "as if its characters were copied into a plain str", but nothing says how to ensure that.
unicodedata.normalize("NFC", text) returns the subclass object itself when the text is already NFC. A following .casefold() then runs the subclass's own override. In my test it returned 'HIJACKED'.
Fix: add a normative sentence such as "The implementation must first obtain a plain str copy without invoking methods overridden by a subclass (for example str.__str__(text))", plus rows R1–R2.
Output depends on the interpreter's Unicode version.
The spec says so in Section 4, but it names no minimum Python version and the examples are fixed.
str.casefold needs Python 3.3 or later, and the U+AB70 row needs Unicode 8.0 or later, which means Python 3.5 or later.
U+1C89 is unassigned in Unicode 15.0, so top_words("\u1c89") returns [] here. As I recall, Unicode 16.0 makes it a letter that folds, so a newer interpreter would give a different answer.
Fix: state the minimum Python version and the Unicode version the examples were verified against, and say that code points unassigned in the running database follow that database.
The interface is under-specified. Section 2 gives no parameter name, while Section 9 calls it text. Fix: write def top_words(text: str) -> list[tuple[str, int]] and say whether the parameter is positional-only.
Two wordings contradict the "\u0345" row.
"A combining mark at the start of the text … is a separator" and "A string containing no letters produces no words" are both false for "\u0345".
The folded-text paragraph resolves this, but the bullets don't say so. Fix: add "in the normalized, case-folded text" to both.
Section 8's preamble uses UserString without importing it. Fix: add from collections import UserString.
Scope versus "any language"

Section 1 admits limits, but the user-facing requirement is "text in any language". Each of these comes from the "separation depends only on General Category" rule. I ran them, and I'm not choosing a behavior for any of them:

Input	Result
Persian می‌خواهم (contains ZWNJ)	می, خواهم
Hebrew צה״ל (gershayim)	צה, ל
Ukrainian п'ять with ASCII apostrophe	п, ять
Catalan col·legi	col, legi
Chinese 二〇二六年 (U+3007 is Nl, so it separates)	二, 二六年
English can't	can, t, so t and s appear in rankings
nº, wowǃ, aˆb (U+00BA, U+01C3 and U+02C6 are letters by category)	one word each
a versus a\ufe0f, Hebrew niqqud, Cyrillic stress marks	different words

The decision for you is whether to keep these as documented limits (add the examples to Section 1) or add join rules. A join rule would mean U+2019, ' or U+00B7 between letters, or ZWNJ/ZWJ, does not separate, similar to UAX #29 word boundaries. Either way, Section 1 should not imply broader coverage than Section 3 delivers.

Proposed rows

I computed these with the reference implementation and checked them by hand.

ID	Input	Expected
R1	class S(str): pass; S("Cat cat")	[("cat", 2)]
R2	A str subclass overriding casefold and __iter__, value "Cat cat"	[("cat", 2)]
R3	"z z z k k j j i i h h g g f f e e d d c c b b a a"	[("z",3),("a",2),("b",2),("c",2),("d",2),("e",2),("f",2),("g",2),("h",2),("i",2)]
R4	"\uff41 \U00010428"	[("\uff41",1),("\U00010428",1)]
R5	"a\u20dd"	[("a\u20dd",1)]
R6	"\u01f0"	[("\u01f0",1)]
R7	"\u0390"	[("\u0390",1)]
R8	"\u0345\u20e5"	[("\u03b9",1)]
Author's response

All 34 rows of Section 8 pass and no expected output was reported as incorrect. I did not change spec.md after this review. The remaining findings concern extra tests, a stated Python/Unicode version, and language-specific word-joining rules that the specification deliberately leaves out (it has no language-specific rules, as Section 1 says).