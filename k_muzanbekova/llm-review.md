# LLM Review Report

**LLM Used:** Google AI for building spec / Claude for  review
**Prompt:** "Review the spec.md for the Top Words Problem and verify if it successfully handles multi-language constraints (English, Russian, Kazakh), text normalization, and edge cases like sorting ties."
![alt text](image-1.png)

# Output From LLM

The spec is solid on the core pipeline (lowercase, split on non-letters, keep in-word hyphens and apostrophes, drop single letters and stop-words, sort by count then alphabetically). But it doesn't yet guarantee one plausible result for Kazakh, for sorting ties, or for real-world news text. I reasoned from the text rather than running an implementation, and I can't see `image.png`, so I haven't checked the program output.

## Gaps that allow more than one plausible result

**1. "Letter" is never defined.** The spec says to replace digits and punctuation with spaces, but doesn't say what counts as a letter. An implementation using `[a-zа-яё]+` passes every row in your table, including the Kazakh one. It would still split `қазақ` into pieces, because қ, ә, ғ, ң, ө, ұ, ү, һ and і fall outside `а-я`. Your Kazakh example doesn't catch this: `және` is a stop-word, so no Kazakh-specific letter survives. I'd define a letter as any character where `str.isalpha()` is true after Unicode NFC normalization. Without NFC, a decomposed `й` (и + combining breve) or an `İ` that lowercases to `i` + a combining dot gets split mid-word. Also decide whether `ё` and `е` are the same word.

**2. "Alphabetical order (A-Z, А-Я)" is ambiguous.** Code-point order and real dictionary order disagree for Kazakh and for `ё`. In the Kazakh alphabet `ә` comes right after `а`, but its code point sorts after `я`. So `әлем` vs `бала` and `ёж` vs `жук` come out in opposite orders depending on the reading. I'd specify code-point order, since it's deterministic and needs no locale library, and say plainly that it isn't dictionary order. Your `ПРИВЕТ HELLO` row already implies Latin sorts before Cyrillic.

**3. Real news text isn't covered.**
- The curly apostrophe `’` is very common in published English. As written, `don’t` becomes `don` plus a dropped `t`, so it never merges with `don't`.
- En and em dashes (`Москва—столица`) need to be stated as separators, not hyphens.
- NBSP (common in Russian typography) and soft hyphens or zero-width characters are also unaddressed.
- The cleaner rule is: everything that isn't a letter, or an in-word hyphen or apostrophe, becomes a space. That also settles `_`, `/`, `%`, emoji, and unusual Unicode digits such as `٣`.

**4. The stop-list is global, not per-language.** The program description says "English stop-words," but the list is trilingual. Because there's no language detection, a Kazakh stop-word can delete a legitimate Russian word: `бар` (bar) and `осы` (wasps) are both Kazakh stop-words, and `Бен` (Ben, the name) is filtered because Kazakh `бен` means "and". You could accept this and document it, or switch to per-language lists. The spec should say that stop-words match whole tokens only (`to-do` survives) and that there's no stemming (`cat` ≠ `cats`). Stemming matters for Kazakh, which is agglutinative.

**5. Smaller underspecified points.**
- Is "single letter" counted in letters or characters? Literally, `a-b` has two letters and survives.
- Order of operations: digits become spaces before the hyphen test, which makes `foo-1bar` → `foo`, `bar`.
- `n` ≤ 0, non-int `n`, and non-string `text` are unspecified.
- State that the output has no leading or trailing whitespace.

## Audit of the example table

- `apple 456-apple` doesn't separate "digits → space" from "digits deleted", since both give `apple: 2`. `abc123def` does.
- No row exercises `n`, truncation at the cut, or a tie straddling the cut.
- No Russian stop-word or Russian hyphen example (`какой-то` appears in the prose only).
- Typo: `cat:1` is missing a space. `""` should be described as the empty string, not two characters.

## Suggested additions

| Input | Expected | Pins down |
|---|---|---|
| `abc123def` | abc: 1, def: 1 | digits become spaces, not deleted |
| `Қазақстан қазақ ҚАЗАҚ` | қазақ: 2, қазақстан: 1 | Kazakh letters are letters; case-folding |
| `әлем бала` | бала: 1, әлем: 1 | code-point tie order (dictionary order would reverse it) |
| `ёж жук` | жук: 1, ёж: 1 | same, for `ё` |
| `don't don’t` | don't: 2 | curly apostrophe normalized |
| `Москва—столица` | москва: 1, столица: 1 | em dash is a separator |
| `какой-то день` | день: 1, какой-то: 1 | whole-token stop-words; Russian hyphen |
| `бар осы` | `""` | consequence of the global stop-list (accept or change) |
| `cat cats` | cat: 1, cats: 1 | no stemming |
| `a-b` | a-b: 1 | single-letter rule applies to letters, not characters |
| `top_words("dog cat bird", 2)` | bird: 1, cat: 1 | tie at the cut-off |
| `top_words("cat", 0)` | `""` (or raise `ValueError`, your call) | invalid `n` |

## Housekeeping

- The signature has a stray `•`.
- `DEFAULT\_STOP\_WORDS` has escaped underscores.
- The `image.png` placeholder is broken.
- The example table sits above the "Specification" heading and should live inside it.
- The English list omits very common words (`be`, `not`, `his`, `from`), and the Russian list omits `это` and `вы`. Fine if intentional, but a trending box will be dominated by whatever is missing.

If you'd like, I can fold all of this into a revised spec with a numbered processing pipeline.