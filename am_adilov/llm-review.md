# LLM review of `spec.md`

Prompting **Claude Opus 5** (1M-context variant, model id `claude-opus-5[1m]`,
run as a Claude Code subagent at default effort) with the prompt quoted below
produced the following output; strikethrough indicates LLM output that I deemed
incorrect or irrelevant.

The reviewer was a fresh instance with no knowledge of how the spec was
written. It was allowed to run `python3` to check code points, casefolding and
sort order, and to implement the rules and compare against the examples, but
not to edit the spec.

**This is the last review performed, and its verdict is that the spec has no
significant issues.** It is review round 4. Earlier rounds: round 1 raised 18
findings on the first draft, round 2 raised 11, round 3 raised 3. Each round's
findings were addressed before the next was run; the progression is visible in
this branch's commit history.

Round 4 closes with four minor factual corrections that it states do not change
the resulting program. I accepted all four and applied them in the same commit
as this file, so the spec here differs from the one round 4 read in exactly
those four wordings — the dotless-`I` limitation in rule 13, the three-way
split of rule 13's casefold parenthetical, naming the eleven-way-tie row in
rule 17, and two interface wordings. My notes below each item record that.

## The prompt

> Review the specification at
> `/Users/adilovamir/kbtu/csda/literate-programming-github-2026/am_adilov/spec.md`:
> find errors, ambiguities, underspecified behavior, and places where the
> examples contradict the rules.
>
> The spec answers this problem statement from a course e-book:
>
> "A news site shows a 'Trending words' box beside each article. Write a Python
> function that finds the 10 most common words in an article's text, along with
> how many times each one appears. Articles may be in English, Russian, or
> Kazakh. Python 3. Name the function top_words and put it in top_words.py."
>
> This is a later review round; earlier reviews have already been addressed, so
> expect the spec to be tight. Judge it as a specification an implementer must
> be able to follow to exactly one program.
>
> Read the spec file, then write your review as markdown prose. Verify the
> examples against the rules by hand — in particular check Unicode code point
> ordering claims, casefold behavior, and whether each stated expected output
> actually follows from the rules as written. You may run python3 to check code
> points, casefold results and sort order, and to implement the rules and
> compare; do not edit the spec.
>
> Report only issues that would actually cause two competent implementers to
> write different programs, or that are factually wrong. For each, give
> numbered findings naming the rule or example, what is wrong, and a concrete
> fix. Do not pad the list with stylistic preferences, speculative edge cases
> that cannot arise in English, Russian or Kazakh news text, or suggestions to
> document things the rules already determine. If the spec has no significant
> issues, say exactly that and explain what you verified to reach that
> conclusion.
>
> Return ONLY the review markdown as your final message — it will be pasted
> into a file verbatim. Do not add a preamble, and do not summarize what you
> did.

---

# Review: `am_adilov/spec.md` — "Top words"

**Verdict: the spec has no significant issues.** I could not find a point at which two competent implementers following these rules would produce different programs, and every factual Unicode claim I checked holds on CPython 3.13 / Unicode 15.1.0. The four items at the end are real but editorial: none of them changes the resulting program, and I am listing them only because they are inaccurate as written, not as blockers.

## What I verified

I transcribed rules 1–20 into a reference implementation and ran every row of the examples table against it, plus the "Worked tie-break" section and the `TypeError` row: **all 38 rows reproduce exactly**, including `[("b", 2), ("cat", 1)]` for `"<b>cat</b>"`, `[("do", 2), ("re", 2)]` for both `"re--do re do"` and `"re-'do re do"`, and `[("n", 2)]` for `"ŉ ʼn"`.

Code point and ordering claims, all confirmed: U+002D (45) is below `ы` U+044B, so `"кот" < "кот-а" < "коты"` and the three-way tie in that row is ordered as printed; `у` U+0443 < `ү` U+04AF; `'` U+0027 < `’` U+2019; `l` U+006C is below both U+02BB and U+02BC, so `"qala"` really does precede `"qaʼla"`/`"qaʻla"`; `"hyphen" < "soft"`; `"мы"` (U+043C) < `"с"` (U+0441) < `"тобой"` (U+0442); and the eleven-way `a`…`k` tie keeps `a`–`j` and drops `k`.

Casefold behavior, all confirmed: `"İ".casefold()` is `"i" + U+0307`, NFC does **not** recompose that pair, so the second mark-deletion pass in rule 1 is what produces `internet`; `"ŉ".casefold()` is `U+02BC + "n"`; `"ß".casefold()` is `"ss"`; `"Ё"` casefolds to `ё` and only then meets rule 14's fold, which is exactly what the `"ЁЖ ёж"` row pins. I also checked that no code point anywhere casefolds to a combining mark that NFC then recomposes into a different letter, so the step-5/step-6 ordering (normalize, then delete) can never bite beyond the `İ` case.

Rule 5's identity claim is exactly true: I compared `ch.isalpha()` against general category `L*` for all 0x110000 code points with zero disagreements. The rule's warning about `re` is also load-bearing in a useful way — the obvious shortcut `[^\W\d_]` would admit `No`/`Nl` characters such as `½`, and rule 5 heads that off.

Rule 3 is precisely right: over the Russian and Kazakh Cyrillic alphabets in both cases, exactly `ё`/`Ё` and `й`/`Й` have canonical decompositions, and all nine Kazakh-specific letters have an empty decomposition.

Most importantly for tightness, I exhaustively checked the casefold/joiner interaction, because that is where rules 9 and 14 could have left holes. Across all code points, U+0149 is the *only* character whose casefold begins with a joiner, none ends with one, and no character's casefold introduces anything that is neither alphabetic, a joiner, nor `Mn`/`Mc`. So the three questions the rules do not spell out — whether a word must be re-split when casefolding reintroduces an interior doubled joiner, whether joiner trimming iterates or runs once, and whether rule 14's "folded form is empty" clause can fire — are *vacuous* rather than ambiguous. They cannot arise from any input, in any script.

Finally: lone surrogates pass through `unicodedata.normalize`, `str.casefold` and `unicodedata.category` without raising on CPython 3.13, so the interface's "no other exception" promise and the `"\ud800"` → `[]` row are both achievable by the straightforward implementation (I tested several surrogate placements, including adjacent to letters and to combining marks). `unicodedata.unidata_version` on 3.13 is indeed `15.1.0`, and none of the characters used in the examples changed category, case folding or decomposition between Unicode 14.0 and 15.1, so the portability caveat in the interface section is accurate.

## Minor factual corrections (none affect the program)

1. **Rule 13's Kazakh Latin rationale is self-defeating for one of the two letters it names.** The rule justifies casefolding with "Kazakh Latin has both `I ı` and `İ i`, and sentences start with capitals." Casefold handles `İ` → `i` as claimed, but `"I".casefold()` is `"i"`, not `"ı"` — so a sentence-initial Kazakh Latin word written with dotless `I` folds to an `i`-initial word, while its mid-sentence occurrences written with `ı` fold to `ı`. The same word is therefore counted as two entries, which is the precise failure the sentence claims to fix, and it also merges `ı`-words with unrelated `i`-words. The rules are still unambiguous, so this does not change what an implementer writes — but the stated reason is wrong. Fix: drop the `I ı` half of the parenthetical and keep only the `İ i` motivation (which casefold does deliver), and add one sentence stating the known limitation, e.g. "Turkic dotless `I` folds to `i`, so a sentence-initial dotless-`I` word does not group with its `ı` forms; a Turkic-aware fold would need language detection and is out of scope." The alternative (mapping U+0049 → U+0131) must *not* be specified, since it would break English `I`.

   *[Accepted and applied, including the recommendation not to remap U+0049. Rule 13 now states the dotless-`I` limitation outright instead of implying the opposite.]*

2. **Rule 13's parenthetical mislabels two of its three examples.** It says casefolding can "emit characters that cannot occur in a word (`İ` → `i` + U+0307, `ŉ` → `ʼ` + `n`, `ß` → `ss`)". Only U+0307 cannot occur in a word. U+02BC demonstrably can — it is a joiner by rule 6 and appears inside the word `qaʼla` two rows down in the examples table — and `s` obviously can. Fix: "…can change a word's length (`ß` → `ss`), emit a combining mark, which cannot occur in a word (`İ` → `i` + U+0307), and emit a joiner in a position a word cannot start with (`ŉ` → `ʼ` + `n`)."

   *[Accepted and applied verbatim. My one sentence had claimed three different things about three different characters.]*

3. **Rule 17 points at the wrong row.** "the eleven-way tie of the last example row" — the `a`…`k` row is fourth from last; the last row is the `42` / `TypeError` row. Fix: "in the eleven-way tie of the `"a b c d e f g h i j k"` row".

   *[Accepted and applied. The row stopped being last when later findings added rows after it.]*

4. **Interface wording.** "A `str` subclass is accepted, and its keys come back as plain `str`" — the function returns no keys; say "the words in the returned pairs are plain `str`". (The behavior itself is already forced: both `unicodedata.normalize` and slicing a `str` subclass yield plain `str`.) Separately, "The function needs Python 3.11 or later" is unsupported by anything in the spec — the annotations need only 3.9 and nothing else in the rules is version-gated apart from the Unicode data, which the surrounding sentences already cover.

   *[Accepted and applied, both halves. The 3.11 floor was left over from an earlier draft that had no conformance target; the Unicode database is the only thing version-dependent, and the bullet already covers it.]*

---

## How the spec was checked, independently of the review

Each round's example table was also verified mechanically: a script parses the
table out of `spec.md`, evaluates each input cell as a Python literal, runs it
through a reference implementation of the rules, and compares with the expected
output cell. All 38 rows match on CPython 3.13, and the `TypeError` row is
checked separately. This is what caught, in an earlier round, that the
invisible characters the examples depend on (U+00AD, U+0301, U+0306, U+0335,
U+2011) had been written as literal characters rather than `\u` escapes.

## Round history

| Round | Findings | What it caught |
| --- | --- | --- |
| 1 | 18 | NFC rationale factually wrong (the Kazakh letters named have no decomposition); `Unicode Alphabetic` unreachable from the stdlib; multiple joiners per word ambiguous; no numbered rule for the cut to ten |
| 2 | 11 | "longest match at each position" permits overlapping words (`"cat"` → `at`, `cat`, `t`); Russian stress marks shredding words, dismissed as malformed input; the returned spelling contradicted between two rules |
| 3 | 3 | a stale cross-reference in the interface; "a joiner sorts below any letter" false for U+02BB–U+02BD; rule 4 silent on single-pass versus iterative mark deletion |
| 4 | 0 significant | verdict above, plus the four editorial corrections |
