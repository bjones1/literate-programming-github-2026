# LLM review: Top words

## Model and reviewed version

- Model: **DeepSeekV4-pro**, as identified by ChatGPT in this conversation.
- Review date: **2026-10-24**.
- Reviewed document: spec.md

## Exact prompt

> Evaluate this spec file. Rate this spec file from 1 to 10, explain complexity and edge cases. Give feedback on the spec in general.

## Full LLM review output

Spec Evaluation: top_words Trending Words
Rating: 9/10
Complexity Assessment: High
This spec is deceptively complex. It describes what appears to be a simple function (top_words(text) -> str) but the implementation touches:

Concern	Complexity
Multi-script Unicode handling	English + Russian + Kazakh alphabets, NFC/NFD normalization, combining marks, variation selectors, surrogate pairs, NUL bytes
Error taxonomy	5 exception classes with hierarchy, precedence rules, structured attributes (.kind, .index, .offenders, .length)
Pipeline architecture	4-stage pipeline (validate → normalize → count → format) with strict data-flow contracts
Algorithmic	Min-heap with inverted lexicographic comparator, O(N log k) bound
Testing	Unit + integration + property-based (Hypothesis) + performance smoke test
The spec covers ~150 lines of actual implementation across ~2,000 lines of specification. That's an unusually high spec-to-code ratio, justified by the Unicode/script-handling requirements.

Edge Cases Covered (Extensive)
Normalization:

Empty/whitespace/punctuation-only strings

Foreign letters (é, ñ, ü, ß) → separators, not errors

NFC composition (и\u0306 → й) and NFC-induced expansions (a\u0301 → á → separator)

NBSP, thin space, emoji, digits, underscores, apostrophes as separators

Kazakh-only letters (ә ғ қ ң ө ұ ү һ і) handled distinctly from Russian (а-я, ё)

Validation:

NUL bytes, lone surrogates, type errors, config errors

Error precedence: TypeError > config ValueError > TextTooLongError > MalformedUnicodeError > UnsupportedScriptError > EmptyArticleTextError

Uncomposable combining marks on supported letters (q\u0307 → rejected) vs. composable (и\u0306 → accepted)

Mixed valid/invalid input ("Привет, naïve" → rejected, not partially accepted)

Counting & Formatting:

k <= 0, k >= U, tie-breaking at k-boundary (lexicographically smallest wins)

Duplicate (word, count) entries preserved verbatim in formatting

Invariants:

Idempotence, length monotonicity, no-empty-tokens, determinism, gate property (normalize never raises post-validation)

Strengths
Byte-for-byte acceptance tests (§9) are the gold standard. test_top_words_exact_output pins down exact output strings, eliminating "close enough" implementations.

Precedence rules are contract, not implementation detail (§6.3). Testing that "naïve\x00" raises MalformedUnicodeError (not UnsupportedScriptError) prevents ad-hoc error ordering.

Property-based invariants (§10) catch edge cases hand-written tests miss — e.g., test_validate_normalize_agreement validates the architectural claim that validation only changes policy, not words.

No ambiguity about top_words composition (§9.1): it must be exactly format(top_k(normalize(validate(text)), 10)), not a reimplementation.

Design decisions are documented with rationale (§12): why normalize strips foreign letters rather than raising, why "don't" → "don t", why digits split rather than glue.

Performance bound is specified (§14): O(N log k), with a slow test guarding against O(N²) regressions.

Integration guidance (§10 of error-handling): HTTP status mapping, logging rules (never log raw article text), batch-job behavior.

Weaknesses
Spec includes the complete solution. The reference implementations for every component (normalize, validate_article_text, top_k_frequent, format_frequency_list) are provided in full. This blurs the line between specification and implementation guide — an implementer can copy-paste rather than reason. If this is a coding assignment, consider removing the reference implementations and keeping only the test vectors.

Min-heap with ReverseLexWord is over-engineered for k=10. The spec explicitly rejects sorted(freq.items(), key=lambda x: (-x[1], x[0]))[:k] in favor of a heap with an inverted comparator class. At k=10, the O(U log U) sort is faster in practice due to CPython's C-level Timsort. The heap optimization matters for k=10,000, not k=10. This adds ~20 lines of complexity for negligible benefit.

test_normalize_length_monotone is risky. The invariant len(normalize(s)) <= len(s) holds for practical inputs but could theoretically fail for Unicode characters where NFC expands (e.g., U+0344 → two marks). Since expanded characters become separators and collapse to nothing, it likely holds — but this is a proof-by-cases that Hypothesis might falsify on exotic input. Consider weakening to len(n) <= 2 * len(s) or documenting the exception.

No stopword filtering discussion. "Trending words" for a news site almost always filters common function words (the, a, is, и, в). The spec's own examples show "the: 3" as valid output — which is correct per the spec, but arguably wrong for the product. A sentence in §1 acknowledging "stopword filtering is out of scope" would prevent confusion.

max_length semantics are subtle. The limit is checked on the raw input, not the NFC form. Since NFC can change length, len(validate(text)) != len(text) is possible. This is documented (§4) but easy to miss. A test vector showing NFC-expanded input near the boundary would help.

Spec is ~2,000 lines for ~150 lines of code. This is verbose even by rigorous standards. Much of the test code is redundant with the reference implementation (e.g., §5.1's parametrized vectors could be derived from §6 of the normalization section). Consider compressing by cross-referencing.

Missing Edge Cases
Gap	Why It Matters
Long single-token input (e.g., 1MB single word)	Tokenization via split() handles this, but memory profiling isn't tested
Words containing :	The output format "word: N" would be ambiguous if a word could contain :. Since words are alphabet-only, this is safe — but the spec should note it (it does in a comment, §11 test_repeated_same_input_deterministic, but not as a formal invariant)
Locale-dependent lowercasing	str.lower() is locale-independent in Python 3, but Turkish İ → i̇ (two chars) is a known trap. The spec's filtering-before-lowercasing order handles this, but a test vector would confirm
Variation selector U+E0100–E01EF (supplementary plane)	Mentioned in §2 of the test layout as intentionally untested, but no explanation of why it's safe to skip
Overall Feedback
This is an excellent spec for a coding assignment, particularly if the goal is to teach:

Unicode-aware text processing

Error taxonomy design with precedence

Property-based testing with Hypothesis

Pipeline architecture with clear data-flow contracts

For a production spec, it's over-specified:

The reference implementations should be removed

The heap algorithm should be relaxed to "any correct O(N log U) or O(N log k) approach"

Stopword filtering should be explicitly listed as out-of-scope or added as a requirement

For a test suite, it's nearly perfect:

Byte-for-byte acceptance tests are exactly what's needed for a contract

Precedence tests prevent error-ordering drift

Property tests catch what hand-written vectors miss

Biggest single improvement: Replace the ReverseLexWord heap with sorted(...)[:k] and note that this satisfies O(N log U) ≤ O(N log N). The heap adds cognitive load for no practical benefit at k=10.

Second-biggest: Add a "non-goals" section explicitly listing stopword filtering, stemming, lemmatization, and cross-article aggregation as out of scope — so implementers don't wonder why "the" appears in the output.