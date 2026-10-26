# LLM review of spec.md

Prompting Claude Sonnet 5.5 (claude.ai chat, default settings) with the full
text of the final `spec.md` pasted in and a request to review it for errors,
ambiguities, and untested decisions produced the following output. I also had
it write a reference implementation of the spec and run it against the
Examples table. Strikethrough indicates LLM output that I deemed incorrect or
irrelevant.

## Minor

1. **A combining mark at the start of a word stays in the word.** Because
   category `M` characters count as letters (Decision 1), `"\u0301a"` is the
   word `"\u0301a"`, which differs from `"a"`. Decision 2 only discards runs
   made *entirely* of marks. Likewise, `"a-\u0301"` is a single word, since a
   connector between a letter and a mark is kept. This is consistent with the
   rules and unlikely in real text, but one sentence or an example would remove
   any doubt.
2. **Only one example uses Kazakh.** Kazakh is one of the three required
   languages, but only `Қазақстан қазақ ҚАЗАҚ` covers it. Consider adding
   `"Ә ә Ң ң Ғ ғ Ү ү Ұ ұ Һ һ Ө ө І і"`, which should return eight entries with
   count 2, ordered by code point (`і`, `ғ`, `ң`, `ү`, `ұ`, `һ`, `ә`, `ө`).
   This also exercises casefolding of capital `І` (U+0406). I checked this
   output against the reference implementation.
3. **The exception message is unspecified.** Decision 9 fixes the exception
   type only. Add a sentence saying the message is not part of the
   specification.
4. **Redundant rule.** "A hyphen followed by a line break splits words" in
   Decision 2 already follows from the rule that a trailing connector splits
   words. It is harmless, and the example documents it.
5. **Unicode version dependence.** `isalpha()`, NFC, and `casefold()` results
   depend on the Python interpreter's Unicode database. Only very rare
   characters are affected.
6. ~~The function should be pure and not mutate its argument.~~ `str` is
   immutable and the spec describes only a return value, so this adds nothing.
7. ~~`True` is redundant in the `TypeError` example, since `bool` is not a
   subclass of `str`.~~ It is redundant but documents that `bool` is rejected;
   no change needed.

## Verified as correct

All 31 rows of the Examples table, the processing order, the tie-breaking rule
(ascending code point), the Python 3.9 requirement for
`list[tuple[str, int]]`, and the handling of `ß`, `ς`, `İ`, `ё`, and the
Cyrillic `і` versus Latin `i`.