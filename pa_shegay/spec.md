## Specification: `top_words`

**File:** `top_words.py`. **Python:** 3.9+, standard library only.

**Signature:** `top_words(text: str) -> list[tuple[str, int]]`

Returns up to 10 `(word, count)` pairs, sorted as in rule 6. The input is the article's plain text (no HTML parsing).

### Rules

1. **Word definition.** A word is a maximal run of Unicode letters (`str.isalpha()`), where an apostrophe (`'` or `’`) or hyphen (`-`) _between two letters_ is part of the word. Digits, punctuation, emoji, and symbols are separators. Words made of digits or mixed letters and digits are not counted.
2. **Case.** Words are lowercased with `str.casefold()`. Counts are merged across case, and the returned word is the casefolded form.
3. **Normalization.** Input is NFC-normalized first. `’` is converted to `'` before tokenizing. `ё` is **not** merged with `е`. Kazakh letters (ә, ғ, қ, ң, ө, ұ, ү, һ, і) are ordinary letters. Casefolding is plain `casefold()`, so Turkish-style `İ`/`I` rules are not applied.
4. **No stemming, no stop words.** "run" and "running" are different words. "the", "и", "және" are counted like any other word.
5. **Script mixing.** Tokens are not split by script. A letter run mixing Latin and Cyrillic is one word.
6. **Order.** Sort by count descending. Ties are broken alphabetically by code point of the casefolded word, ascending.
7. **Fewer than 10 distinct words.** Return all of them. Empty text or no words returns `[]`.
8. **Invalid input.** A non-`str` argument raises `TypeError`.
9. **Single-letter words** (e.g. "я", "a") count.

### Examples

| #   | Rule                    | Input                                              | Expected output                                                                        |
| --- | ----------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 1   | Basic                   | `cat dog cat bird dog cat`                         | `[("cat",3),("dog",2),("bird",1)]`                                                     |
| 2   | Case                    | `The the THE`                                      | `[("the",3)]`                                                                          |
| 3   | Case (Cyrillic)         | `Привет привет ПРИВЕТ`                             | `[("привет",3)]`                                                                       |
| 4   | Punctuation             | `Hi, hi! hi?`                                      | `[("hi",3)]`                                                                           |
| 5   | Apostrophe inside       | `don't Don’t dont`                                 | `[("don't",2),("dont",1)]`                                                             |
| 6   | Edge apostrophe         | `'cat' cat`                                        | `[("cat",2)]`                                                                          |
| 7   | Hyphen inside           | `well-known well known`                            | `[("well-known",1),("known",1),("well",1)]`                                            |
| 8   | Edge hyphen/dash        | `cat - cat -- dog`                                 | `[("cat",2),("dog",1)]`                                                                |
| 9   | Digits                  | `2024 abc123 cat`                                  | `[("cat",1)]`                                                                          |
| 10  | Kazakh letters          | `Қазақстан қазақстан Ұлы`                          | `[("қазақстан",2),("ұлы",1)]`                                                          |
| 11  | ё vs е                  | `ёж еж`                                            | `[("ёж",1),("еж",1)]` (tie → code point order: `е` U+0435 < `ё` U+0451, so `еж` first) |
| 12  | No stemming             | `run running runs`                                 | `[("run",1),("running",1),("runs",1)]`                                                 |
| 13  | Stop words kept         | `и и и дом`                                        | `[("и",3),("дом",1)]`                                                                  |
| 14  | Mixed script            | `привет hello привет`                              | `[("привет",2),("hello",1)]`                                                           |
| 15  | Script mix in one token | `aб aб`                                            | `[("aб",2)]` (Latin "a" + Cyrillic "б" is one word)                                    |
| 16  | Tie-break               | `b a c a b c`                                      | `[("a",2),("b",2),("c",2)]`                                                            |
| 17  | Top-10 cutoff           | 11 distinct words, counts 11 down to 1             | first 10 only; lowest word dropped                                                     |
| 18  | Tie at cutoff           | 10 words appear twice, `zzz` once, plus `aaa` once | the ten 2-count words; neither 1-count word appears                                    |
| 19  | Fewer than 10           | `one two`                                          | `[("one",1),("two",1)]`                                                                |
| 20  | Empty / no words        | `""`, `"123 !!!"`                                  | `[]`                                                                                   |
| 21  | Single letters          | `я a я`                                            | `[("я",2),("a",1)]`                                                                    |
| 22  | Normalization           | `é` (e + combining accent) twice, once precomposed | `[("é",3)]`                                                                            |
| 23  | Newlines/whitespace     | `cat\ncat\t cat`                                   | `[("cat",3)]`                                                                          |
| 24  | Bad type                | `None`                                             | raises `TypeError`                                                                     |
