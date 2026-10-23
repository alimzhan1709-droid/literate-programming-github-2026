# LLM Review of `spec.md`

Prompting Claude Sonnet 5 (via Claude Code, run as a fresh subagent with no
memory of drafting the spec, so this is a genuine independent review rather
than the same context re-approving its own work), with:

> You are reviewing a specification for a university assignment (CSDA 5101,
> Advanced Software Paradigms). Read the file at this path: [spec.md].
> Context: the original, under-specified problem statement this spec is
> tightening is: "A news site shows a 'Trending words' box beside each
> article. Write a Python function that finds the 10 most common words in
> an article's text, along with how many times each one appears. Articles
> may be in English, Russian, or Kazakh." The goal of the spec is to leave
> only ONE plausible implementation — i.e., resolve every ambiguity,
> omission, implicit assumption, interface-contract question, and boundary
> case the one-paragraph original leaves open. Review spec.md against that
> original prompt to check whether it succeeds at that goal. Specifically
> check: 1) Multilingual correctness (English/Russian/Kazakh) — is the word
> definition actually sound for all three scripts? 2) Tie-breaking and
> ranking — fully deterministic? 3) Edge cases — empty input, fewer than 10
> words, non-string input. 4) Anything in the spec that is still ambiguous,
> internally inconsistent, or where a reasonable second implementer could
> produce a different result than intended. 5) Anything missing that the
> original prompt implies but the spec never addresses. Do not read or run
> top_words.py or check_tables.py in that directory — review spec.md on its
> own merits, as a human reviewer would without seeing any implementation.
> Give a numbered list of findings. For each finding, say whether it's a
> real gap or just a stylistic nitpick. Be genuinely critical.

produced the following output; strikethrough indicates LLM output that I
deemed incorrect or irrelevant.

---

Reviewed spec.md on its own merits. Overall it is unusually careful — the
word-definition regex, case-folding, stop-word, and tie-break decisions are
all cross-checked against their own example tables, and every example I
hand-verified ... computes out exactly as the spec claims. Tie-breaking
(count desc, then casefolded word ascending by code point) gives a genuine
total order with no residual ties possible ... so ranking is fully
deterministic. Edge cases (empty string, digit/punctuation-only, <10
distinct words, non-str input) are all explicitly and consistently covered.

Findings:

1. **Real gap — no decision on inflectional normalization.** Russian and
   Kazakh are heavily inflected; "дом/дома/дому/домом" ... are, by this
   spec, four distinct words. The spec explicitly argues against stop-word
   removal ... but never applies the same reasoning to
   lemmatization/stemming, even though it's an equally plausible, equally
   consequential reading of "most common words" for these two languages
   specifically — and the original prompt calls out multilingual support
   as a core requirement. ... Should have been decision #7.

   **Applied.** Added as decision 4 ("Inflectional forms are not
   normalized"), with its own disambiguation table, verified against the
   implementation.

2. **Nitpick (technical imprecision).** The spec asserts `[^\W\d_]` is
   "exactly a letter" in Python's Unicode regex. Not quite: Python's `\w`
   is alnum-or-underscore, so `[^\W\d_]` also matches non-decimal numeric
   chars (Roman numerals, vulgar fractions, superscripts — Unicode
   categories Nl/No) that aren't letters per `str.isalpha()`. Unlikely to
   matter for real news text in these three scripts, but the "exactly"
   claim is false as stated.

   **Applied.** Verified empirically (`'Ⅳ'.isalpha()` is `False` but it
   matches `[^\W\d_]`); reworded the spec to state the distinction
   precisely instead of claiming equivalence.

3. **Minor real gap.** NFC normalization is motivated as fixing
   precomposed-vs-decomposed Cyrillic, which it does for the Kazakh-specific
   letters .... But if input ever contains a base letter plus a combining
   mark with no NFC precomposed form, that mark isn't itself alnum and
   would act as an invisible word-splitting delimiter — a residual case the
   spec's own reasoning about normalization doesn't quite close off.
   Narrow, but real.

   **Applied.** Verified empirically (Cyrillic а + combining macron has no
   NFC precomposed form and does split into two tokens). Added as an
   explicit, accepted Limitation rather than extending the tokenizer,
   since combining diacritics don't occur in ordinary news text in these
   three languages.

4. **Nitpick (presentation).** Section 1's example table already shows
   casefolded, lowercase output ... before decision #2 (case folding) is
   introduced — a reader going top-to-bottom hits output casing that isn't
   yet justified. The document is internally consistent once read in full,
   but the ordering invites confusion.

   **Applied.** Added a one-sentence forward reference in decision 1 rather
   than reordering the whole document.

5. **Nitpick.** No explicit decision on minimum word length — single-letter
   tokens ("и", "в", "I", "a") are counted as words purely as a side effect
   of the general letter-run rule, never spotlighted as a choice ... despite
   being just as plausible a point of disagreement.

   **Applied.** Added one clarifying sentence to decision 1; didn't give it
   a full decision of its own, since — unlike stop words — there's no
   plausible alternative reading to weigh against, just an implication of
   the general rule worth stating outright.

6. **Nitpick.** Section 1's example cells mix prose ... with the "word:
   count" format used everywhere else, slightly obscuring whether listed
   word order there also reflects the tie-break rule (it does, verified).

   **Applied.** Reformatted the `covid19` row to the same `word: count`
   style as every other row.

7. ~~**Very minor omission.** No statement of what "text" assumes about
   upstream input (plain text vs. HTML-bearing extraction) — likely out of
   scope given the original prompt doesn't raise it either, but it's the
   kind of "downstream producer/implicit context" question the course's own
   taxonomy flags.~~

   **Rejected.** The interface section already defines `text` as "the
   article's full text" — i.e. already-extracted plain text, the same
   framing the original one-paragraph prompt uses. HTML extraction is a
   different function's problem entirely; specifying it here would be
   solving a problem the assignment never poses. The reviewer's own hedge
   ("likely out of scope") agrees.

Verdict: strong spec, closer to achieving "one plausible implementation"
than most student drafts would be, but item 1 was a genuine, non-cosmetic
hole given the course's explicit multilingual-correctness criterion.

---

After applying findings 1–6, I re-ran every example in `spec.md`'s tables
against a fresh implementation written from the spec alone (not from memory
of writing the spec) — all pass, including the three ordering mistakes I'd
made by hand in the original tables (`"Hello world!..."`, `"don't stop
state-of-the-art"`, `"well--known"`) and a fourth I caught the same way
while adding the new inflectional-forms example (`"дом дома дому домом"`).
Cyrillic alphabetical ordering by Unicode code point is not something to
compute by hand — every ranked example in this spec is now verified by
running code, not by eye.
