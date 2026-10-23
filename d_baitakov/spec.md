# Specification: `top_words`

## Problem

A news site shows a "Trending words" box beside each article. Write a Python 3
function `top_words` in `top_words.py` that finds the 10 most common words in
an article's text, along with how many times each one appears. Articles may be
in English, Russian, or Kazakh.

## Definitions

**Input.** `top_words(text: str) -> list[tuple[str, int]]`. The input is a
single string of already-extracted article text. `None` is not accepted
(passing anything other than `str` is a caller error; the function does not
have to handle it).

**Word.** A **word** is a maximal substring of `text` consisting of Unicode
letters (Unicode general categories `Lu`, `Ll`, `Lt`, `Lm`, `Lo`), optionally
joined into one word by apostrophes (`'` U+0027 or `’` U+2019) appearing
between letters. Everything else — digits, whitespace, punctuation, hyphens,
underscores — is a separator and terminates a word. A candidate that contains
no letter (e.g. `"19"`, `"42"`, `"---"`) is not a word and is discarded.

Consequences of this definition, each deliberate:

- Digits split words: `covid19` is the word `covid` followed by the
  non-word `19`.
- Hyphens split words: `state-of-the-art` is four words.
- Underscores split words: `foo_bar` is `foo` and `bar`.
- An internal apostrophe joins: `don't` is the one word `don't` (apostrophe
  kept, exactly as written). An apostrophe at the start or end of a run cannot
  join anything, so `'tis` is the word `tis`.
- The apostrophe is kept in the output word; `don't` and `dont` are distinct
  words. The two joining characters are not interchangeable with each other
  either: `don't` (U+0027) and `don’t` (U+2019) are two different words.

**Case normalization.** Each word is normalized with Python's
`str.casefold()` before counting. `Cat`, `cat`, and `CAT` are one word, `cat`.
Casefold is used rather than `lower()` because it also folds the German `ß`
and similar entries; for Cyrillic and Latin-based Kazakh text it behaves like
lowercasing. No other normalization is performed:

- `ё` and `е` are distinct words (`ёж` ≠ `еж`).
- Accented and unaccented Latin letters are distinct (`café` ≠ `cafe`).
- No stemming, no stop-word removal. There is no stop-word list in this
  specification, so `the`, `и`, `және` count like any other word. (The
  "Trending words" framing could suggest filtering stop words; this spec
  deliberately does not — with no agreed list, filtering would not produce
  one plausible result.)
- No Unicode normalization. Since only general categories `L*` are word
  characters, a combining mark (category `Mn`) separates words: the
  decomposed `éclair` is the two words `e` and `clair`. Article text is
  assumed to be in NFC form (as published text normally is), under which
  `éclair` is the one word `éclair`.

**Count.** The count of a word is the number of times its casefolded form
occurs in `text`.

**Output.** A list of `(word, count)` tuples, where `word` is the casefolded
word and `count` its occurrence count. The list contains at most 10 tuples:

1. Ordered by count, highest first.
2. Words with equal counts are ordered lexicographically by `word` (Python's
   default string comparison, i.e. by Unicode code point). This applies both
   to ordering within the list and to which words survive the cut: if the
   10th and 11th words have equal counts, the lexicographically smaller one
   is kept.
3. If the text contains fewer than 10 distinct words, all of them are
   returned; if it contains no words at all, the result is `[]`.

The number 10 is fixed by the problem statement and is not a parameter.

## Examples

Each example below distinguishes this spec from at least one other plausible
reading of the original problem.

| Input | Expected output |
|---|---|
| `cat dog cat bird dog cat` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| `Cat cat CAT` | `[("cat", 3)]` — case is folded |
| `don't don't dont` | `[("don't", 2), ("dont", 1)]` — internal apostrophe joins; `don't` ≠ `dont` |
| `'tis 'tis` | `[("tis", 2)]` — a leading apostrophe does not join |
| `state-of-the-art` | `[("art", 1), ("of", 1), ("state", 1), ("the", 1)]` — hyphens split; the 1–1 ties break by code-point order |
| `covid19 covid covid` | `[("covid", 3)]` — digits split; the letterless `19` is discarded |
| `foo_bar foo` | `[("foo", 2), ("bar", 1)]` — underscore splits, so `foo_bar` contributes one `foo` |
| `42 42 ---` | `[]` — no words at all |
| `Кот кот КОТ` | `[("кот", 3)]` — Cyrillic casefolds too |
| `ёж ёж еж` | `[("ёж", 2), ("еж", 1)]` — `ё` is not folded to `е` |
| `Мен мен және Мен` | `[("мен", 3), ("және", 1)]` — Kazakh words count; no stop-word removal |
| `don't don’t` | `[("don't", 1), ("don’t", 1)]` — U+0027 and U+2019 are different joining characters; `don't` sorts before `don’t` by code point |
| `"éclair éclair"` | `[("clair", 1), ("e", 1), ("éclair", 1)]` — no NFC normalization; the decomposed form splits at the combining mark |
| `b a c a b c d` | `[("a", 2), ("b", 2), ("c", 2), ("d", 1)]` — equal counts sort lexicographically |
| `k j i h g f e d c b a` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` — eleven 1-count words tie at the cut; `k` is dropped because `j < k` |
| `hello world hello` | `[("hello", 2), ("world", 1)]` — fewer than 10 distinct words returns all of them |
| `""` | `[]` |
