# LLM review

Assistant: OpenAI Codex  
Model (as reported by the user): GPT-6.1 Sol  
Reasoning effort (as reported by the user): Max

## Prompt used

Review spec.md for completeness, consistency, and feasibility. Identify ambiguous requirements, contradictions, missing edge cases, and untestable acceptance criteria.

The assistant revised the specification and then reviewed the resulting version in the same conversation. This review applies to the corrected `spec.md` delivered alongside this file.

## Latest review output

### Result

No unresolved contradictions or missing word-processing rules were identified in the revised specification.

The specification defines the input contract, processing order, word boundaries, normalization, counting, sorting, output format, and behavior at the ten-item boundary.

### Findings from the previous review

1. **Word recognition is now explicit.**

   Words begin with a Unicode letter and continue through letters and marks. Other characters act as separators. Leading marks are ignored.

   Examples cover punctuation, apostrophes, hyphens, dashes, digits, underscores, whitespace, attached marks, and mixed scripts.

2. **Case normalization is now explicit.**

   Each extracted word is normalized using `str.lower()`. Returned words use this normalized representation.

   The `"Straße STRASSE"` example distinguishes lowercasing from case folding.

3. **Unicode normalization is now explicit.**

   NFC normalization is applied before word extraction. Canonically equivalent representations are merged.

   Separate examples cover composed and decomposed Latin and Cyrillic characters, preservation of attached marks, and handling of leading marks.

4. **The ten-word limit now has a valid example.**

   The specification includes an input with 11 unique words of equal frequency. The expected result contains exactly the first 10 words in Unicode lexical order.

   This also defines selection when frequencies are equal at the cutoff.

5. **Expected outputs now match the declared return type.**

   All successful examples use lists of `(str, int)` tuples. Invalid-input examples explicitly require `TypeError`.

6. **Examples now cover the stated processing decisions.**

   The examples distinguish word forms, accented and unaccented words, different scripts, Unicode ordering, stop-word inclusion, and single-letter word inclusion.

### Executed verification

A temporary Python reference implementation was written from the revised specification and executed against every example in its table.

Environment:

- Python 3.12.14.
- Unicode database version 15.0.0.

Results:

- 24 valid-string examples produced their exact expected outputs.
- 3 invalid-input examples raised `TypeError`.
- All 27 checks passed.

The verification used a temporary reference implementation. The submitted files remain `spec.md` and `llm-review.md`.

### Review conclusion

The revised specification is sufficiently precise to implement and test using the stated Python Unicode operations. No further corrections were identified during this review.