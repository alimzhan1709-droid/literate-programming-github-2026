# LLM review of `spec.md`

## Model and disclosure

Model: **GPT-6, through Codex**. The assistant that edited the specification
also performed this review. This is a self-review, not an independent review
by Claude. A more specific model identifier and the reasoning-effort setting
are not available to report.

## Response to the instructor's review

1. Added scope wording explaining that accepting scripts does not provide
   linguistic segmentation. Added examples for Japanese text without spaces
   and Devanagari vowel marks separating letters.
2. Explicitly accepted subclasses of `str` using `isinstance(text, str)` and
   required returned words to be plain `str`. Added a subclass acceptance
   example outside the table.
3. Set the minimum version to Python 3.10 and documented possible Unicode
   differences between runtimes and how to reproduce results.
4. Added a note that the Kazakh example follows Unicode code point order,
   not Kazakh alphabetical order.
5. Removed the redundant sentence about separators never joining words.
6. Checked the title: the local file already has exactly one space after `#`,
   so no title edit was needed.

Also added all suggested optional examples: Greek final sigma, text without
spaces, and rejection of `bool` and `bytearray`. The table explicitly notes
that the `bytearray` input uses a constructor expression rather than a literal.
The second NFC pass, the `İX` example, and the existing cutoff and prefix
ordering rules are retained.

## Last review

Prompt used for this GPT-6 self-review, with the complete revised `spec.md`
and the instructor's feedback in context:

> Review the revised spec.md for ambiguity, contradictory requirements,
> incorrect expected results, and missing coverage. Check every numbered
> issue and optional example in the instructor's review. Specifically check
> script segmentation and combining marks, str subclasses and plain str
> outputs, Python and Unicode version requirements, Kazakh code point order,
> redundant wording, the title, Greek sigma, and invalid bool/bytearray inputs.
> Check that NFC, casefold, NFC, isalpha tokenization, counting, sorting, and
> the ten-word cutoff remain consistent. Report actionable defects separately
> from limitations of verification. Do not modify spec.md during this review.

Review output:

> No actionable defects found in the revised specification.
>
> All six numbered issues are addressed. Scope now distinguishes acceptance
> of alphabetic characters from linguistic word segmentation. Text without
> separators remains a maximal alphabetic run, and surviving combining marks
> separate words. The Japanese and Devanagari examples illustrate both cases.
>
> Input subclasses are explicitly accepted, while returned words must be
> plain str. The separate Article subclass example checks both requirements.
> Python 3.10 is the minimum; runtime Unicode data is authoritative, and the
> text explains possible version-dependent results and reproducibility.
>
> The Kazakh tie order is explicitly identified as code point order. The
> redundant separator sentence is removed. The heading has one space after
> its Markdown marker. Greek final sigma merges with ordinary sigma as
> expected, and bool and bytearray inputs explicitly raise TypeError.
>
> The five transformation and counting steps remain consistent with the
> acceptance results. The second NFC pass is retained. The İX example still
> splits at the combining dot introduced by case folding. Ranking remains
> descending frequency followed by ascending code point order, with prefix
> ordering and a strict ten-entry cutoff.
>
> Verification limitation: the temporary reference implementation passed
> all 39 table examples and the separate subclass example on Python 3.14.0
> with Unicode data 16.0.0. This supports consistency of the examples with
> the rules; it does not test every input or establish compatibility by
> execution on every supported Python version. No submitted implementation
> was tested because this submission contains a specification and review.

No review findings were rejected; no strikethrough is needed.

## Execution check

A temporary implementation followed the five steps literally. A checker
parsed the table's literal inputs and expected results with `ast.literal_eval`,
constructed the explicitly noted `bytearray` input separately, and checked
both return values and required exceptions. It also checked the subclass
example and plain output word types.

```text
Python: 3.14.0
Unicode: 16.0.0
Checked 39 table examples and 1 subclass example; 0 failures
```

The temporary implementation and checker were executed outside the submitted
files. This review does not imply instructor approval.
