LLM review of `spec.md`
=======================

Model and disclosure
--------------------

Last review model: **DeepSeek V4.1-Flash-1017**

Two earlier reviews, described below, were performed before it: the first by
Claude Sonnet 5.5 and the second by DeepSeek V4.1-Flash-1017. Their findings
were applied to `spec.md` before the last review was run.

Revision before the final review
--------------------------------

### First review (Claude Sonnet 5.5)

The first review traced the examples against the rules and reported that they
all held, with three problems. Its output was:

> I traced every example against the rules by hand and they all hold, so the fix
> is small. I found three problems:
>
> 1. Stray markup at the end. The document ends with `<p><br></p>`, which
>    doesn't belong in a spec. I removed it.
> 2. `bool` is not covered. `True` is a number type in Python, and the spec
>    lists "numbers" without pinning this down. I added an example row.
> 3. Apostrophe lookalikes are not covered. Only `'` and `’` are treated as
>    apostrophes. `ʼ` (U+02BC) passes `isalpha()`, so it's a letter, while `‘`
>    (U+2018) is a separator. I added a note and an example.

These three changes were applied to `spec.md` before the second review.

### Second review (DeepSeek V4.1-Flash-1017)

The second review was run with the prompt `review spec.md`. Its output was:

> Overall, `spec.md` is strong: the algorithm is mostly deterministic, the
> ranking rules are clear, and the examples pin down many tricky cases. I did
> not find incorrect expected values in the table. The main issues are a few
> ambiguities and one overbroad non-goal statement.
>
> Findings
> --------
>
> 1. **~~Apostrophe grammar is slightly ambiguous~~**
>
>    * "Inside a run, a single apostrophe may join two letters" does not
>      explicitly say whether multiple apostrophes are allowed when each one is
>      individually between letters.
>
>    * E.g. should `"rock'n'roll"` or `"a'b'c"` be one word? The intended answer
>      is almost certainly yes, but the spec should state it formally.
>
>    * Suggested normative wording: after casefold/NFC and U+2019 replacement, a
>      word is a maximal substring matching:
>
>      ```text
>      letter+ (?: ' letter+ )*
>      ```
>
>      where `letter` is any character with `str.isalpha() == True`.
>
>    * This also makes `"a''b"` split into `a` and `b`, and leading/trailing
>      apostrophes separators.
>
>    * Add examples for `"rock'n'roll"` and `"a'b'c"`.

> 2. **"No merging of distinct letters" is too absolute**
>
>    * This conflicts with casefold behavior. `Straße`/`STRASSE` already merge
>      via casefold, and casefold can also map e.g. Kelvin sign `K` to `k`.
>    * More subtly, `İ`.casefold() produces `i` plus a combining dot; under the
>      spec's leftover-combining-mark rule, that combining mark is a separator,
>      so `İ` and `i` can both become `i`.
>    * Suggested rewording: "No additional merging beyond casefold and NFC: no
>      NFKC, no ё→е, no қ→к, no accent stripping, etc."
>    * Consider adding a test for `İ i` if the current behavior is intended.

> 3. **~~`str` subclass behavior is not fully pinned down~~**
>
>    * The spec accepts `str` subclasses, but says `text.casefold()`.
>    * A subclass can override `casefold()`, so exact behavior depends on
>      whether the implementation calls `text.casefold()` or
>      `str.casefold(text)`.
>    * Either specify "use `str.casefold(text)` to ignore overrides" or
>      explicitly say overrides are honored.
>    * Add a simple subclass test, e.g. `class S(str): pass;
>      top_words(S("cat")) == [("cat", 1)]`.

> 4. **~~Missing test for U+02BC~~**
>
>    * The spec explicitly says `ʼ` U+02BC is a letter, but there is no example.
>
>    * Add something like:
>
>      ```python
>      "don't donʼt" -> [("don't", 1), ("donʼt", 1)]
>      ```
>
>      This pins that U+02BC stays a letter and is not treated as an apostrophe.

> 5. **Minor wording improvements**
>
>    * "A word is a maximal run of letters" is a bit misleading because
>      apostrophes can be inside the word. The formal grammar above would remove
>      that ambiguity.
>    * "leftover combining marks" is fine, but could say "after NFC, any
>      remaining combining mark is a separator."
>    * The type-error rules are clear; no change needed there.
>
> Verdict
> -------
>
> The spec is implementable and the examples are strong. Before freezing it,
> clarify the apostrophe grammar, reword the "no merging" non-goal, and specify
> `str`-subclass/casefold behavior. Add the U+02BC and multiple-apostrophe
> tests.

The instructor's feedback on my pull request required items 1 and 2 of this
review to be addressed. These changes were applied to `spec.md`:

* Item 1: step 4 now states the word grammar as `letter+ ( ' letter+ )*`, with
  new examples `"rock'n'roll"` and `"a'b'c"`.
* Item 2: the "no merging" non-goal now reads "No merging beyond `casefold()`
  and NFC", with a new `'İ i'` example.
* The U+02BC example from item 4 was added.

Item 3 (`str` subclasses) and the remaining minor wording in item 5 were not
applied.

Last review
-----------

Prompt used with DeepSeek V4.1-Flash-1017, with the complete revised `spec.md`
as context:

> review spec.md

Review output (strikethrough marks output that I deemed incorrect or
irrelevant):

> Overall: the spec is strong and implementable. I checked the examples against
> the stated rules and found no mismatches or contradictions. The ranking,
> apostrophe handling, Unicode normalization order, and cutoff behavior are all
> pinned down well. Findings are minor clarification issues rather than
> blockers.
>
> Findings
> --------
>
> | Severity | Finding |
> |---|---|
> | Minor | “~~Straight apostrophe `'`” should explicitly say **U+0027**. The spec already gives U+2019 for curly; giving both code points removes font ambiguity~~. |
> | Minor | ~~The word grammar `letter+ ( ' letter+ )*` uses spaces and a bare apostrophe. It would be clearer as: let `A = U+0027`; a word matches `letter+ (A letter+)*`.~~ |
> | Minor | “~~Casefold itself can merge characters” is misleading for `İ`. Casefold **expands/decomposes** `İ` into `i` + U+0307, which then remains a separator after NFC. Better wording: “casefold can expand one character into multiple code points; those code points are then classified as letters or separators by `isalpha()`.~~” |
> | Minor | Since `str` subclasses are explicitly accepted, `text.casefold()` could be overridden by a subclass. Specify that subclasses are treated as their underlying string content, e.g. use `str.casefold(text)` or state “subclass overrides are ignored.” |
> | Minor | Ranking is clear, but could be made fully mechanical: “sort distinct words by `(-count, word)` and take the first 10.” This removes any prose ambiguity around ties at the cutoff. |
> | Optional | Consider one extra example mixing U+2019 and U+02BC, e.g. `"a’b aʼb"`, to pin that U+2019 joins after replacement while U+02BC is a letter. Not required, but cheap coverage. |
>
> Suggested edits
> ---------------
>
> 1. Change:
>
>    > Replace every curly apostrophe `’` (U+2019) with a straight apostrophe
>    > `'`.
>
>    to:
>
>    > Replace every U+2019 RIGHT SINGLE QUOTATION MARK with U+0027 APOSTROPHE.
>
> 2. Change the word definition to something like:
>
>    > Let `A` be U+0027. A word is a maximal substring matching
>    > `letter+ (A letter+)*`, where `letter` is any character for which
>    > `str.isalpha()` is true after steps 1–2.
>
> 3. Rephrase the casefold note:
>
>    > Casefold can expand a character into multiple code points. For example,
>    > `ß` becomes `ss`, and `İ` becomes `i` plus U+0307 COMBINING DOT ABOVE; NFC
>    > does not compose that sequence, so U+0307 is a separator under the
>    > `isalpha()` rule.
>
> 4. Add a subclass clause:
>
>    > For `str` subclasses, process the underlying string value; overridden
>    > methods on the subclass do not change the result.
>
> No blocking issues. The example table is consistent with the rules, including
> the tricky `İ`, `x\u0301y`, `ʼ`, `Ａ`, mixed-script, and cutoff-tie cases.

Implementation check
--------------------

Using the `spec.md` that was reviewed above, Claude Sonnet 5.5 wrote a temporary
Python implementation (NFC after `casefold()`, `isalpha()` tokenization with the
inner-apostrophe rule, counting, then sorting by descending count and ascending
code point, truncated to 10). A separate script parsed every input and expected
result from the Markdown examples table and called the implementation for each
row, including the `TypeError` rows.

Execution result on Python 3.12.3 with Unicode data 15.0.0:

```text
Checked 38 examples; 0 failures
```

The temporary implementation and checker are kept outside this submission; the
assignment's two submitted files are `spec.md` and `llm-review.md`.
