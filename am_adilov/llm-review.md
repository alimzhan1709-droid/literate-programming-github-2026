# LLM review of `spec.md`

**LLM used:** Claude Opus 5 (1M-context variant, model id `claude-opus-5[1m]`),
running as a Claude Code subagent at default effort. The reviewer was a fresh
instance with no knowledge of how the spec was written; it was allowed to run
`python3` to check code points, casefolding and sort order, and to implement
the rules and compare, but not to edit the spec.

**Rounds:** two. Round 1 reviewed the first draft and raised 18 findings — a
factually wrong rationale (the Kazakh letters it claimed NFC would repair have
no decomposition), an unreachable definition (`Unicode Alphabetic` is not
available from the standard library), unresolved ambiguity about multiple
joiners per word, and a missing numbered rule for the cut to ten. I addressed
those and re-ran the review. **The round 2 review is reproduced below — it is
the last review performed.**

**Prompt used, both rounds** (verbatim, with a sentence added in round 2 to say
it was a second round):

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
> This is a second review round; an earlier review has already been addressed,
> so expect the spec to be fairly tight. Judge it as a specification an
> implementer must be able to follow to exactly one program.
>
> Read the spec file, then write your review as markdown prose with numbered
> findings. For each finding give: the rule or example number it concerns, what
> is wrong or unresolved, and a concrete suggestion. Verify the examples against
> the rules by hand — in particular check Unicode code point ordering claims,
> casefold behavior, and whether each stated expected output actually follows
> from the rules as written. You may run python3 to check code points, casefold
> results and sort order, and to implement the rules and compare; do not edit
> the spec. If you find nothing substantive in some area, say so plainly rather
> than padding the list.
>
> Return ONLY the review markdown as your final message — it will be pasted
> into a file verbatim. Do not add a preamble, and do not summarize what you
> did.

**How to read what follows:** the review is reproduced verbatim.
~~Strikethrough~~ marks LLM output I judged incorrect or irrelevant. Notes in
*[square brackets and italics]* are mine, not the LLM's; "numbers in the review
refer to the rules of the spec as it stood at review time", which is the
version in the commit's parent, since I renumbered the rules while addressing
the findings.

---

# Review: `top_words` specification (round 2)

I implemented the rules as written (NFC → split → casefold → re-NFC → `ё`-fold → count → sort → cut) on CPython 3.13 / UCD 15.1 and ran all 30 example rows plus the worked tie-break. **All 30 rows reproduce exactly as printed**, including every ordering claim (`у` U+0443 < `ү` U+04AF, `кот` < `кот-а` < `коты` with U+002D below `ы`, `don't` U+0027 < `don’t` U+2019, `qala` < `qaʼla` because U+006C < U+02BC, `й` U+0439 < `о` U+043E). The invisible characters the examples depend on are really present in the file: row 127 is `и`+U+0306 vs composed `й`, row 128 is `о`+U+0335+`й`, row 129 is U+2011, row 130 is U+00AD. Rule 3's factual claims hold (`й` = 0438 0306, `ё` = 0435 0308; none of the nine Kazakh-specific Cyrillic letters has a decomposition), rule 4's gloss is exact (I verified `isalpha()` ⇔ general category `L*` over all 0x110000 code points, zero mismatches), and rule 1's "re-normalize to NFC" step is genuinely load-bearing (24 alphabetic characters casefold into a decomposed form that NFC recomposes, e.g. `ǰ` U+01F0 → `j`+U+030C → U+01F0). So the findings below are about rules whose consequences are unstated, not about broken examples.

## 1. Rule 6 — "the longest match at each position" permits overlapping words

As written, the rule quantifies over *every* position: "A **word** is the longest match of `A+(J A+)*` at each position, scanning left to right." At position 0 of `"cat"` the longest match is `cat`, but position 1 also has a longest match (`at`) and position 2 has `t`. A literal implementer gets `[("at",1),("cat",1),("t",1)]` for `"cat"`. Every example in the table silently assumes matches are non-overlapping and that scanning resumes *after* the match, but no rule says so — and this is the single most load-bearing rule in the spec.

**Suggestion:** restate as a scan: "Scan left to right. At each position, if a match of `A+(J A+)*` starts there, take the longest such match as a word and resume scanning at the first character after it; otherwise advance one character." I fuzz-tested this reading (200k random strings over letters, the four joiners, space, digit, period) against `re.findall(r'A+(?:J A+)*')` with greedy quantifiers and got zero mismatches, so once non-overlap is stated the rule is operationally unambiguous and an implementer may use a single regex.

*[Accepted — this was the worst defect in the draft. Now rule 7, restated as a scan that resumes after each match, with `"cat"` as the first example row.]*

## 2. Rules 9 — Russian stress marks are not "malformed", and the rule shreds them

Rule 9's only illustration of a surviving combining mark is `о` + U+0335 as a "hand-rolled `ө`", and the rule dismisses the whole class: "Such input is malformed, and the caller normalizes it upstream." But the most common surviving mark in real Russian text is U+0301 COMBINING ACUTE ACCENT, used for stress (`мо́лодец`, `бо́льшая` vs `больша́я`), which appears in news copy, headlines and quoted dictionary forms. I verified that NFC composes U+0301 with **no** Cyrillic vowel (`а е и о у ы э я` all stay as base + U+0301), so `"мо́лодец"` is counted as the two words `мо` and `лодец` — a word is destroyed and two junk words enter the trending box. That is not malformed input and the caller has no NFC-based way to "normalize it upstream"; it would have to strip marks, which is exactly the decision the spec is dodging.

**Suggestion:** make an explicit decision and show it. Either (a) add a pipeline step that drops category `Mn`/`Mc` characters inside a word instead of treating them as separators (so `мо́лодец` → `молодец`), or (b) keep the current behavior but replace the "malformed" sentence with a named, exampled case: a row `"мо́лодец молодец"` → `[("лодец", 1), ("молодец", 1), ("мо", 1)]` and a sentence saying stress-marked Russian is accepted as such. Option (a) also disposes of finding 4 below.

*[Accepted, option (a) — now rule 4. Stress-marked Russian counts as the unmarked word, and as predicted this also fixed finding 4.]*

## 3. Rules 4–5 — the joiner set is one character wide where it needs to be several

Rule 5 excludes U+02BC from the alphabetic class "so that it behaves like the apostrophe it is", but U+02BB MODIFIER LETTER TURNED COMMA, U+02BD, ~~U+02B9~~ and the rest of the Lm apostrophes are left alphabetic by rule 4. The result is that two visually identical marks behave in opposite ways, verified:

- `"qaʼla ʼqala"` → `[("qala", 1), ("qaʼla", 1)]` (the documented row)
- `"qaʻla ʻqala"` → `[("qaʻla", 1), ("ʻqala", 1)]` — U+02BB is a *letter*, so the leading one is kept, and the key `ʻqala` sorts after every Latin word because U+02BB (0x2BB) > `z` (0x7A).

U+02BB is not exotic in this domain: it is the standard Uzbek/Karakalpak `oʻ`, `gʻ` character, it is routinely substituted for U+02BC by editors and input methods, and Kazakh Latin text copied from Turkic sources carries it. Nothing in rule 5's rationale explains stopping at one code point.

**Suggestion:** either widen the joiner set to the apostrophe-like modifier letters (at minimum U+02BB and U+02BD alongside U+02BC) and say so with a row, or add one sentence to rule 5 stating deliberately that U+02BC is the only modifier letter excluded and that U+02BB/U+02BD count as letters inside words — with a row pinning it, since this is the kind of thing two implementers will guess differently.

*[Accepted for U+02BB and U+02BD, with a `"qaʻla ʻqala"` row. I struck U+02B9: it is MODIFIER LETTER PRIME, used for palatalization and as a prime symbol, not an apostrophe — including it would merge genuinely different characters.]*

## 4. Rules 11 + 12 — casefolding `İ` is named but its consequences for Kazakh Latin are not

Rule 11 gives `İ` → `i` + U+0307 as a curiosity and says "the result is the key regardless". Two consequences go unstated, and both matter because the spec itself puts Kazakh Latin in scope (rule 5, and the `qaʼla` row), and the 2018 alphabet has both `I ı` and `İ i`:

- `"İnternet internet"` → `[("internet", 1), ("i̇nternet", 1)]`. Every sentence-initial `İ`-word is a separate trending word from its lowercase form, forever. For a Kazakh Latin article this is not an edge case, it is every sentence.
- `"Iı ıı II"` → `[("ii", 1), ("iı", 1), ("ıı", 1)]`: `I` folds to dotted `i`, merging the dotless/dotted distinction in the other direction.

Rule 12's "This is the only *language-specific* folding" does technically decide the question (no Turkic casing), but a reader will not connect that sentence to the `İ` aside three lines earlier. Separately, the key `i̇nternet` contains U+0307 — a combining mark, which rule 9 declares a separator, i.e. a character that by construction can never occur inside a word. The character model and the key model disagree, with nothing saying that is intended.

**Suggestion:** add a sentence to rule 11 spelling out that `İ`-initial words do not merge with their lowercase forms and that `I` folds to dotted `i` (so dotless `ı` stays distinct), plus one example row, e.g. `"İnternet internet"` → `[("internet", 1), ("i̇nternet", 1)]`. State explicitly that a key may contain characters that could not appear in a word. If finding 2's option (a) is adopted, note that it also removes U+0307 here and makes `İnternet` merge.

*[Accepted, and resolved in the better direction: because rule 4 now deletes combining marks after casefolding too, `İnternet` groups with `internet` instead of standing alone. The example row records that.]*

## 5. Rules 7 + 11 — casefolding can put a *joiner* at the start of a key, contradicting rule 7

Rule 7 reads as a structural guarantee: "a leading or trailing joiner is not part of the word". That holds for the split, but not for the returned key, because casefolding runs afterwards. I scanned all alphabetic code points for ones whose casefold (after NFC) contains a joiner: there is exactly one, U+0149 `ŉ`, which casefolds to U+02BC + `n`. Verified: `"ŉ ʼn"` → `[("n", 1), ("ʼn", 1)]` — the single-letter word `ŉ` becomes the key `ʼn` with a leading joiner, while the input `ʼn` becomes `n`, so the two spellings are distinct trending words and the `ŉ` key sorts after all Latin words.

This is only one character, and the current rules do determine the answer, so the fix is cheap — but as written rule 7 states a false invariant about outputs.

**Suggestion:** qualify rule 7 ("…is not part of the *split* word; casefolding may reintroduce one — see rule 11"), and add one clause to rule 11 saying whether leading/trailing joiners are re-trimmed after folding. If you choose to re-trim, say it in rule 1's step list, since it adds a pipeline step. Either decision is fine; silence means two implementers differ on `ŉ`.

*[Accepted. `ŉ` is an archaic Dutch character that will never appear in an English, Russian or Kazakh article, so the finding is irrelevant to the problem domain — but the fix costs one clause, and a false invariant in a spec is worth removing regardless. Joiners are now trimmed after folding (rule 14), with a row.]*

## 6. Rule 11 — "the casefolded spelling is what the function returns" is contradicted by rule 12

Rule 11 states flatly that the returned key is the casefolded spelling. Rule 12 then replaces U+0451 with U+0435, and the row `"ещё еще"` → `[("еще", 2)]` returns a spelling that is neither input word's casefold. Rule 1's step order resolves it, but a reader who takes rule 11 at its word and reads rule 12 as affecting only *grouping* ("so `ещё` and `еще` are one word") could return `ещё` as the display form with count 2 — which is arguably the better product behavior, and is exactly why the ambiguity is worth closing in words rather than only in an example.

**Suggestion:** change rule 11 to "…the casefolded spelling, after the `ё`-fold of rule 12 and the NFC pass of rule 1, is what the function returns", and add to rule 12 "the returned spelling is the folded one, `еще`, even if `ещё` is the only spelling that occurs in the text."

*[Accepted verbatim — rule 14 now says exactly that.]*

## 7. Rules 14 and 15 — one claim is example-bound, another misses the boundary

Rule 14 says: "With eleven words tied at one occurrence, which ten appear is fully determined: `k` is the one dropped, on every machine." As a general rule this is simply false — `k` is dropped only for the `a…k` input of the last table row. The rule reads as if it were stating a property of eleven-way ties. Separately, "on every machine" sits awkwardly against rule 17's admission that results depend on the interpreter's Unicode database (see finding 8).

Rule 15's "Return every word when the article has fewer than ten distinct words" leaves the boundary unstated: with exactly ten distinct words you also return all of them. Nothing breaks — rule 14 covers it — but the sentence as phrased does not.

**Suggestion:** rule 14: "Keep the first ten pairs and discard the rest. Ties are cut by rule 13's order, not by input order, so when more words tie than there is room for, which of them survive is still fully determined — in the eleven-way tie of the last example row, `k` is the word dropped." Rule 15: "ten or fewer distinct words".

*[Accepted, both wordings, now rules 17 and 18.]*

## 8. Rule 17 vs. the Interface — the conformance target is not pinned

The Interface permits "Python 3.11 or later"; rule 17 admits the result "depends only on the Unicode database of the running interpreter, which differs across Python versions"; the examples preamble says they were produced on CPython 3.13. Taken together, the spec does not determine one program: two conforming implementations on 3.11 (UCD 14.0) and 3.14 (UCD 16.0) can disagree, and the reader has no stated basis for deciding which is correct. In practice none of the 30 rows is at risk — every character they use has been assigned for decades and I reproduced all of them on UCD 15.1 — but that is a fact about the examples, not something the spec says.

**Suggestion:** add to rule 17 or the Interface: "Conformance is judged on CPython 3.13 (`unicodedata.unidata_version` 15.1.0), which is what the examples were produced on. On other versions, results may differ only for characters whose category, case folding or decomposition changed; no example in this document uses such a character." That turns a loose end into a checkable statement.

*[Accepted, moved into the Interface section.]*

## 9. Interface — the `TypeError` test is unspecified for `str` subclasses

"Raise `TypeError` if `text` is not a `str`" does not say whether the test is `isinstance(text, str)` or `type(text) is str`. The two differ for `str` subclasses, which do reach functions like this in real code (`enum.StrEnum` members, `pathlib`-adjacent wrappers, ORM string columns). Under `isinstance`, `top_words(MyStr("a b a"))` returns `[("a", 2), ("b", 1)]`; under the strict test it raises. The claim "No other exception is raised: every `str`, including `""`, has an answer" also quietly covers lone surrogates — I checked that `'\ud800'` and `'a\ud800b'` survive `unicodedata.normalize` and `casefold` on CPython 3.13, giving `[]` and `[("a",1),("b",1)]`, so the claim is true, but it is true by luck of the implementation rather than by anything stated.

**Suggestion:** "Raise `TypeError` if `isinstance(text, str)` is false; `str` subclasses are accepted and their keys are returned as plain `str`." Optionally add "including strings containing unpaired surrogates" to the no-other-exception sentence, since that is the one input class a reader would suspect of raising.

*[Accepted, both parts, with rows for the subclass and the lone surrogate.]*

## 10. Examples section — the cells are Python literals, which is never stated

Row 132's input `"cat🙂dog\ncat"` only yields `[("cat", 2), ("dog", 1)]` if `\n` is read as a Python escape; read as the two literal characters in the file, `n` is alphabetic and the answer is `[("cat", 1), ("dog", 1), ("ncat", 1)]`. The table similarly relies on Python-literal reading for `""` and for `42` (an `int`). Row 137 then breaks the convention entirely by giving its output as prose ("the ten pairs …") rather than a literal. And rows 127–130 depend on characters that are invisible or easy to lose in a copy-paste (U+0306, U+0335, U+2011, U+00AD) — they are correct in the file, but a reader cannot tell `"soft­hyphen"` from `"softhyphen"` on screen.

**Suggestion:** add one sentence under the table: "Inputs and outputs are written as Python literals; escapes are escapes." Give row 137 its literal output. And annotate the invisible-character rows in the input cell itself, e.g. `"soft­hyphen"`, `"й й"`, `"о̵й"`, `"cat‑dog"` — ~~keeping the rendered form in the Decisions column if you want it readable~~.

*[Accepted except the last clause: every row that needs an invisible character now writes it as a `\u` escape in the input cell, and the eleven-way tie row spells out its ten pairs. I did not duplicate rendered forms in the Decisions column — the escapes are the authoritative form and a second rendering invites them to drift apart.]*

## 11. Non-normative justifications that do not hold up

Two parentheticals are decoration rather than rules, but a spec should not carry wrong reasoning:

- Rule 5 calls U+02BC "the modifier letter apostrophe of the 2018 Kazakh Latin orthography". The 2018 revision of the Kazakh Latin alphabet is the one that *removed* apostrophes in favour of acute accents and digraphs; the apostrophe-based variant was the October 2017 decree, and its apostrophe was conventionally ASCII U+0027. U+02BC's real currency in this area is as the Turkic `tutuq belgisi` (Uzbek, Karakalpak) and as the typographically correct apostrophe. The decision to treat it as a joiner is good regardless.
- Rule 4 claims that `isalpha()` and the Unicode Alphabetic property "coincide" for Latin, Cyrillic and Kazakh text. `isalpha()` is exactly `L*` (verified), whereas derived Alphabetic additionally includes `Nl` and `Other_Alphabetic` marks, among them combining Latin letters in the U+0363–U+036F range. The difference cannot be observed from the stdlib and never arises in modern orthography, so the claim is harmless but unverifiable as stated.

**Suggestion:** rule 5: "U+02BC, the modifier letter apostrophe used for the apostrophe in Turkic Latin orthographies". Rule 4: drop the comparison, or say "`isalpha()` is exactly general category `L*`; the derived Alphabetic property is slightly wider, but not for any character occurring in modern Latin, Cyrillic or Kazakh orthography."

*[Accepted, both wordings. My 2018 attribution was wrong; the apostrophe alphabet was the 2017 decree.]*

## Areas I checked and found nothing substantive

- **Every example follows from the rules.** 30/30 rows plus the worked tie-break reproduce exactly, including the `TypeError` row. I found no example that contradicts a rule.
- **All ordering claims are correct.** Code-point comparisons in rules 13 and in rows 122, 123, 126, 128, 130, 135, 136 all check out; "lexicographic by code point, prefix first, never locale-sensitive" matches Python's `str` comparison on every pair the spec asserts.
- **Casefold claims are correct.** `İ` → `i`+U+0307 and `ß` → `ss` are right, casefold of `Ё` U+0401 is `ё` U+0451 so rule 1's ordering really does make row 125 work, and no alphabetic character casefolds to the empty string (so no word can vanish into an empty key).
- **Rule 1's step ordering is complete and the re-NFC step is necessary**, not ceremonial — 24 characters exercise it.
- **Rules 8 and 10** (digits, U+2010/U+2011/U+00AD) are unambiguous and the worked explanations of `covid-19` and `covid-19-vaccine` are right.
- **Rules 16 and 2** (no stop words, no HTML stripping) are clear, exampled, and correctly scoped out.
- I found nothing worth raising about performance, memory, or complexity bounds; for article-length input any direct implementation is fine, and the spec is right not to constrain it.

---

## What I did with the review

Of the 11 findings I accepted 11 in substance and rejected two details inside
them (U+02B9 as a joiner, and duplicating rendered characters beside the `\u`
escapes). The spec in this commit reflects all of it: rule 7 is a
non-overlapping scan, rule 4 deletes combining marks inside words, the joiner
set is six characters, the folded spelling is pinned as the returned one, and
the conformance target is CPython 3.13.

Every example row in the revised spec — 37 of them now — was re-checked against
a reference implementation of the revised rules on CPython 3.13, including the
three rows the review's findings added (`"мо́лодец молодец"`,
`"İnternet internet"`, `"ŉ ʼn"`). All 37 match.
