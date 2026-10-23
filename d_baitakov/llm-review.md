# LLM review of `spec.md`

**LLM used:** Claude Code (CLI agent) running GLM (glm-5.3-flash, Z.ai),
in an interactive terminal session, at default (medium) reasoning effort.

**Prompt used:** `Review spec.md for the Top words problem. Find errors,
underspecified behavior, and examples whose expected output is wrong or
ambiguous. For each finding, say whether a plausible alternative reading of
the original problem would produce a different output.`

This file shows the last review performed. Strikethrough indicates review
output I deemed incorrect or irrelevant and did not act on.

---

## Review output

1. **Apostrophe variants.** The spec says internal apostrophes join words but
   lists two joining characters (`'` U+0027 and `’` U+2019) without saying
   whether they interoperate. `don't` and `don’t` could plausibly be one word
   or two. → **Accepted.** Spec now states they are distinct words and pins
   it with the `don't don’t` example (verified: two entries).

2. **Decomposed Unicode.** The spec counts only general categories `L*` as
   word characters, so a combining mark (category `Mn`) splits a word:
   decomposed `éclair` becomes `e` + `clair`. This was true of the definition
   but never stated, and NFC text is the normal case, so a reader could
   assume normalization. → **Accepted.** Spec now states no Unicode
   normalization, notes text is assumed NFC, and pins it with the
   `"éclair éclair"` example.

3. **Example table error: `foo_bar foo`.** The table listed the expected
   output as `[("bar", 1), ("foo", 1)]`, but `foo` occurs twice (once inside
   `foo_bar`). → **Accepted, fixed.** Correct output is
   `[("foo", 2), ("bar", 1)]`. (Found by running every example through a
   reference implementation written directly from the spec.)

4. **Example table error: `éclair éclair` tie order.** First draft
   ordered `éclair` first, but by code point `é` (U+00E9) sorts after
   `c` and `e`. → **Accepted, fixed.** Correct order is
   `[("clair", 1), ("e", 1), ("éclair", 1)]`.

5. ~~The function should filter common stop words ("the", "и", "және"),
   since a "Trending words" box full of "the" is useless in practice.~~
   **Rejected as a spec change.** The original problem provides no stop-word
   list, so filtering would reintroduce exactly the underspecification the
   assignment asks to remove. The spec explicitly documents this position
   instead.

6. ~~Hyphenated compounds like `state-of-the-art` should count as one word,
   since that is how they are used in English.~~ **Rejected.** Both readings
   are plausible, which is precisely why the spec must pick one; the spec
   picks "hyphens split" and pins it with an example. No change needed.

7. ~~The `re` pattern `[^\W\d_]+` does not match all `L*` categories, so the
   implementation contradicts the definition.~~ **Rejected as incorrect.**
   That pattern matches Unicode letters in Python 3's default (Unicode) mode,
   which is what the definition requires; the Cyrillic and Kazakh examples
   confirm it.

## Verification

After acting on findings 1–4, every example in the spec's table was checked
against a reference implementation written directly from the spec
(`re`-based tokenizer, `casefold`, `Counter`, sort by `(-count, word)`,
truncate to 10). All examples, including the two added by this review, match
their expected output.
