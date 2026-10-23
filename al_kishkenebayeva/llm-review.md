Prompting GPT-6-luna, auto balance mode: "Please review the updated toc.md and give your feedback. Write your feedback in llm-review.md."

---

Overall assessment

This is a strong and well-structured specification. The problem statement is clear, the rules are precise, and the included examples cover a wide range of important edge cases: Unicode scripts, apostrophes and hyphen handling, case sensitivity, tie-breaking, output formatting, and performance requirements. It reads like a polished challenge intended to test both correctness and implementation discipline.

What works especially well

- The specification is explicit about the word definition and boundary rules, which is the main source of ambiguity in this kind of task.
- The examples are varied and useful, especially the tie-break and multilingual cases.
- The performance requirement is concrete and reasonable, which helps prevent under-specified or inefficient solutions.
- The output contract is crisp: ordering, formatting, and empty-input behavior are all clearly defined.

Minor suggestions / opportunities for improvement

1. Add one sentence explaining the intended algorithmic strategy.
   The file emphasizes the result but does not explicitly state that a single-pass streaming solution is expected. A brief note like “This can be solved in one pass with O(n) time and O(k) memory” would help guide students toward the intended performance model without being too prescriptive.

2. Clarify whether the function is expected to work only on Python strings or also on any text-like object.
   The requirement says “Evaluate it to get the text argument,” which is clear in context, but a short note that the function receives a Python str and should not mutate or normalize outside of the specified lowercasing would remove any ambiguity.

3. Consider adding one example demonstrating the exact tie-break at the 10th slot.
   The current examples explain tie-breaking generally, but a tiny scenario with exactly 11th/12th words tied for the last included slot could make the cutoff rule even more obvious to readers.

4. Slightly tighten the wording around “hyphen/apostrophe belongs to the word only when it sits directly between two letters.”
   The rule is already correct, but an explicit example like “rock’n’roll” and “don’t” on both sides of the boundary condition would make it easier to verify by eye.

5. Optional: include a compact “summary of the core invariant” section.
   Something like: “Each word is identified as a maximal run of letters, lowercased, counted, and then ranked by frequency, then by first appearance.” This would reinforce the mental model and reduce misinterpretation.

Overall verdict

I would consider this a very good task specification: precise, testable, and thoughtfully scoped. The current version is already strong; the suggestions above are minor refinements rather than corrections. It is likely to produce a clear and fair coding assignment with minimal ambiguity.
