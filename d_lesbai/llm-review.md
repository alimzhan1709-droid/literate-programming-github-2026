# LLM review of spec.md

Prompting Claude Sonnet 5.5 with `Review spec.md` (the final `spec.md` attached) produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

## Result

I checked the spec for contradictions, underspecified behavior and examples that would not distinguish between plausible readings. I found no contradictions and no wrong expected outputs. The remaining remarks are minor:

1. The word definition is deterministic, including for corner cases (combining mark right after a connector, lone combining marks, mixed scripts, apostrophe variants). Every decision in "Processing steps", "Definition of a word" and "Output order" has at least one example in the table whose output would differ under another plausible reading.
2. The pipeline order (NFC → tokenize → apostrophe normalization → `casefold` → NFC → count → sort) is stated explicitly, and the two cases where order matters (`Straße`, `İ`) have examples.
3. ~~Performance requirements for very large articles (streaming, memory limits) are not stated.~~ The assignment is about behavior; a single in-memory pass is fine.
4. ~~Languages written in Arabic, Hebrew or Indic scripts may contain combining marks inside words; the spec should list them.~~ The rule "category `L*` or `M*`" already covers them, so a list would be redundant.
5. ~~Kazakh text in the Latin alphabet (2021 draft) uses `ı`, `ä`, `ö`, `ü`, `ğ`, `ş`, `ń`, `ý`; the spec should name them.~~ They are Unicode letters, covered by the general rule.
6. A tie between `well` and `well-known` is resolved by code point order (prefix first); this is stated and has an example.

## Examples checked against code

A reference implementation written from this spec passes all 38 rows of the examples table (rows are executable test cases in `top_words.py` / `test_top_words.py`).