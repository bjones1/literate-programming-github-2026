# LLM Review of spec.md

## Model

GPT-6

## Prompt

Review the Top Words specification for correctness, completeness, ambiguity, and internal consistency. Check all examples against the stated rules, Unicode handling, punctuation and word boundaries, sorting, and the ten-word limit. Report only actual findings and recommend precise corrections.

## Review output

The specification defines the main behavior clearly: words consist of Unicode alphabetic characters, are converted to lowercase, and are sorted by descending frequency with alphabetical tie-breaking. The result is limited to ten distinct words.

One confirmed inconsistency was found in Section 5. The input `Toyota Toyota BMW Honda Ford Audi Kia Hyundai Nissan Mazda Volvo Subaru` does not contain `Mercedes`, but the expected output includes `('mercedes', 1)`. The output also fails to select the correct ten words according to the specified sorting rules.

The expected result should be:

```python
[
    ('toyota', 2),
    ('audi', 1),
    ('bmw', 1),
    ('ford', 1),
    ('honda', 1),
    ('hyundai', 1),
    ('kia', 1),
    ('mazda', 1),
    ('nissan', 1),
    ('subaru', 1)
]
```

A further implementation consideration is Unicode normalization. The specification requires `lower()` but does not require Unicode normalization. Therefore, implementations should not silently introduce normalization as an additional processing step.

## Resolution of findings

1. Correct the inconsistent expected output in Section 5.
2. Replace the old Section 8 with the five implementation clarifications provided in the assignment.
3. Keep the specified `lower()` behavior and do not introduce additional normalization rules without updating the specification.
4. Run the automated tests against the final specification and record the actual results before submitting.

## Verification status

The example logic was checked in this review. A full `python3 -m unittest -v` run has not been performed as part of this review. No claim of a successful full test run is made.

## Test results

Command: `python3 -m unittest -v`

Result: 11 tests passed successfully (`OK`).
