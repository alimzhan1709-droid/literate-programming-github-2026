# LLM review

Prompting Claude Sonnet 5.5, medium effort, with `Review spec.md` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

## Output

**Errors**

1. Example 7 contradicts rule 6. Under code-point tie-breaking, the three 1-count words sort as `known`, `well`, `well-known`, but the example lists `well-known`, `known`, `well`.
2. Example 11 contradicts itself. The output column lists `ёж` first, but the note says `еж` sorts first. By rule 6, `еж` comes first.
3. Rule 1 contradicts itself. It says digits are separators, which would make `abc123` the word `abc`, yet it also says mixed letter-digit tokens are not counted. Example 9 follows the second reading. Pick one reading and add an input that tells them apart (`abc123 abc`).

**Underspecified behavior**

4. Combining marks (e.g. Russian stress accents, U+0301) fail `isalpha()` even after NFC, so they would split a word. Say whether they are stripped, kept inside the word, or treated as separators.
5. `casefold()` changes some characters (`ß` → `ss`, final `ς` → `σ`), and the casefolded form is what gets returned. Add an example such as `Straße STRASSE` → `[("strasse", 2)]`.
6. Rule 3 never says whether NFC is re-applied after casefolding, which can matter for some characters.
7. "Hyphen" means only ASCII `-`, but the spec doesn't say so. Other dashes (U+2010, U+2011, en dash) and the modifier apostrophe `ʼ` (U+02BC, which passes `isalpha()`) are unaddressed.
8. No example covers consecutive or mixed joiners, such as `a--b` or `a-'b`. Under rule 1 each should split, but an example would pin it down.
9. The underscore is a separator (`snake_case` → two words), but no example shows it.
10. Characters from scripts without spaces (CJK) pass `isalpha()`, so a whole run becomes one "word". State whether that is acceptable.
11. Example 17 is vague ("11 distinct words, counts 11 down to 1"). Give a concrete input.
12. ~~The spec doesn't say whether the function accepts `str` subclasses.~~
13. ~~The Python version is not specified.~~
14. ~~Consider stop-word removal, since a trending box dominated by "the" and "и" is not useful.~~
15. ~~Consider decoding HTML entities such as `&amp;`.~~
16. ~~Kazakh has multiple orthographies (Cyrillic, Latin, Arabic script), so the spec should add per-script rules.~~

**Verdict**

Items 1–3 are errors that would produce wrong tests. Items 4–11 are gaps where two reasonable implementations would return different output.