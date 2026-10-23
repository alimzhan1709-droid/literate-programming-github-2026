# Review of spec.md

Prompting Claude Opus 5.5, extra high effort, with `Review spec.md` (Claude
Code, `claude -p "Review spec.md" --model claude-opus-5-5 --effort xhigh`, with
read-only file tools) produced the following output; strikethrough indicates
LLM output that I deemed incorrect or irrelevant, and each **Note** says what I
did. This was the seventh review round; the earlier rounds found no wrong
examples either, only gaps that I fixed.

The review could not run code, so I ran every table row against a reference
implementation of section 2 on CPython 3.13; all 36 rows pass.

> I found no errors in `spec.md`. All 35 example rows match the Section 2 procedure. The token grammar and the folding rule leave no room for two correct implementations to give different results. I could only find wording and test-coverage issues.
>
> I couldn't run Python in this session because the commands were blocked, so I checked each row by hand against steps 1–5. I also:
> - Searched the file for words that mix Latin and Cyrillic letters (such as a Latin `A` inside `МОСКВА`) and for hidden marks or invisible characters. There are none, and the Kazakh `і` on line 95 really is U+0456.
> - Checked that the Greek row (line 89) actually tests "fold after NFD". With NFD first the word is `α\u0315ι`; folding without NFD gives `αι\u0315`, so the two methods give different results.
> - Confirmed that a Python `re` pattern gives the same tokens as "longest match", because letters/digits, marks and joiners never overlap.
> - Checked from memory, not against a file, that the list of extra deleted characters in step 1 matches Unicode 16.0, and that UAX #29 does break words at U+200B.
>
> ## Should fix
>
> 1. **No row has a joiner right after a mark.** The rules make `"кто\u0301-то"` one word. But the note on row 67 only says "a joiner needs a letter or digit *after* it". Someone who also requires a letter or digit *before* the joiner would split this into `кто́` and `то`, and every current row would still pass. Add a row: `"кто\u0301-то"` → `[("кто\u0301-то", 1)]`, with the note "a mark may come before a joiner".

**Note:** Fixed; I added this row.

> 2. **`spec.md:21`, "The box shows each word in its case-folded form".** Section 1 is a binding section, but plain `token.casefold()` is not the same as step 4. It keeps `’`, `‐` and `‑`, and it skips NFD/NFC. Read literally, it would show `don’t`, which goes against row 65. Change it to "…in the form given by section 2, step 4".

**Note:** Fixed; section 1 now refers to step 4.

> ## Minor
>
> ~~3. **`spec.md:19`, `str` subclasses.** `isinstance` accepts them, but if a subclass overrides `__iter__` or `casefold`, the result depends on how the code is written. Say that a subclass is read as its plain characters and its overridden methods are not called, or say its behavior is unspecified.~~

**Note:** Article text arrives as a plain `str`; subclasses that override string methods are not a realistic input for this function.

> 4. **`spec.md:23`, "Latin-script Kazakh is accepted but miscounted".** This is too broad: only words containing `I`, `ı` or `İ` are affected. Suggest: "…but words with `I`, `ı` or `İ` may be miscounted".
>
> 5. **No test checks that "digit" means any `isdecimal()` digit, not just ASCII 0–9.** Add `"\uff15G \u0663"` → `[("\uff15g", 1)]`. Code that only accepts `0`–`9` would give `[("g", 1)]`.

**Note:** Items 4 and 5 are correct but minor; the rule in section 2 already says `isdecimal()`, and section 1 already says Latin-script Kazakh is not supported. Left as is.

> ~~6. **`spec.md:12`, "`\u` escapes show invisible or look-alike characters".** `\u00b2`, `\u00bd` and `\u1fb3` are neither. Adding "…or characters whose exact code point matters" covers them.~~

**Note:** The sentence explains why escapes are used; it does not forbid using them elsewhere.

> ## Design risk to confirm
>
> ~~In Kazakh web text, people often type Latin `i` instead of `і` because the two look the same. Row 101 deliberately counts them separately. On real articles, that can turn one common word into two entries, each with part of the count. No change is needed if this is intended.~~

**Note:** Intended, as the reviewer says; merging look-alike letters is the kind of guess section 5 rules out.

> Nothing needs cutting; the spec is already short and doesn't repeat itself.
>
> ~~The other folders have an `llm-review.md`; I can save this review there in the same format if you want.~~
