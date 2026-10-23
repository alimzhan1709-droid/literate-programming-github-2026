# Specification: `top_words`

## Interface

```python
def top_words(text: str) -> list[tuple[str, int]]:
    ...
```

- `text`: the article's full text, as a single `str`. It may mix English,
  Russian, and Kazakh in any proportion, including within one sentence.
  Any other type raises `TypeError`.
- Returns: the 10 most frequent words in `text` and their counts, as a list
  of `(word, count)` tuples, ranked most frequent first. If `text` contains
  fewer than 10 distinct words, the list is shorter than 10 — it is never
  padded. If `text` contains no words at all, the list is empty (`[]`).

## Decisions

Each decision below resolves one place where the original one-paragraph
problem statement admits more than one reading. Every table gives an input,
this spec's output, and at least one different output a *different*
plausible reading of the original problem would produce instead.

### 1. What counts as a word

A word is a maximal run of Unicode letters — any character `c` for which
`c.isalpha()` is true — optionally extended across an internal apostrophe or
hyphen when a letter immediately follows it. Formally, the authoritative
definition is this regular expression, applied to the input after Unicode
NFC normalization:

```python
import re, unicodedata

_WORD = re.compile(r"[^\W\d_]+(?:['’\-][^\W\d_]+)*")

def _words(text: str) -> list[str]:
    return _WORD.findall(unicodedata.normalize("NFC", text))
```

`[^\W\d_]` is "a word character that is not a digit and not `_`". In
Python's Unicode-aware `re`, this matches every letter plus a handful of
rare non-decimal numeric characters (Roman numerals like `Ⅳ`, vulgar
fractions like `½`) that `\w` also treats as word characters but
`str.isalpha()` would not; for realistic English/Russian/Kazakh news text
this distinction does not arise, so the implementation below uses the regex
class rather than filtering through `isalpha()` character by character. It
covers Latin, Cyrillic, and the nine extra Kazakh letters (`ә ғ қ ң ө ұ ү һ
і` and their uppercase forms) with no special-casing needed, since they are
ordinary Cyrillic letters as far as Unicode is concerned.

Digits, underscores, whitespace, and all other punctuation are delimiters,
never part of a word. A hyphen or apostrophe (straight `'` or curly `’`) is
*part of* a word only when a letter immediately follows it; a leading,
trailing, or doubled hyphen/apostrophe is just a delimiter instead. There is
no separate minimum word length: a single letter standing alone (English
`a`/`I`, Russian `и`/`в`) is a valid one-letter word under this rule, counted
like any other.

The NFC normalization step matters for Cyrillic text in particular: the same
visible word can arrive as precomposed or decomposed Unicode, and without
normalizing first those two byte sequences would be counted as different
words. (Examples below show words in their casefolded form, per decision 2 —
word *boundaries* are decided before casing is applied, but this document
shows the two together.)

| Input                                    | Expected output (top words)        | A different plausible reading                                                             |
| ----------------------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------------- |
| `"Hello world! Привет мир. Сәлем, әлем."` | `hello: 1, world: 1, мир: 1, привет: 1, сәлем: 1, әлем: 1` | Dropping the Kazakh-only letters by matching only `[a-zA-Zа-яА-Я]` would mangle `сәлем` into `с` + `лем`. |
| `"covid19 won 2026"`                      | `covid: 1, won: 1` (`2026` and the `19` in `covid19` are dropped entirely — digits never form or extend a word) | Treating digits as word characters would instead produce `covid19` and `2026` as words.      |
| `"don't stop state-of-the-art"`           | `don't: 1, state-of-the-art: 1, stop: 1` | Splitting on every apostrophe/hyphen would instead produce `don`, `t`, `state`, `of`, `the`, `art`. |
| `"well--known"`                           | `known: 1, well: 1`                 | Treating a doubled hyphen the same as a single one would instead produce `well-known` as one word. |

**Limitation (accepted, not solved here):** a base letter followed by a
*combining* diacritic mark (Unicode category Mn) that has no precomposed
NFC form — e.g. Cyrillic `а` + combining macron, which has no single
precomposed codepoint — is not reassembled by NFC normalization. The
combining mark itself does not match the word-character class above, so it
acts as an invisible delimiter and splits what a reader would see as one
word into two. This is accepted as out of scope: combining diacritics do
not occur in ordinary English/Russian/Kazakh news text, only in specialized
phonetic or linguistic transcription.

### 2. Case folding

Every word is folded with Python's `str.casefold()` before counting, and the
output words are the casefolded form. `casefold()` rather than `lower()`
specifically because it also normalizes cases `lower()` misses (e.g. German
`ß`); for English/Russian/Kazakh text the two usually agree, but `casefold`
is the more correct and no more expensive choice.

| Input                | Expected output | A different plausible reading                          |
| --------------------- | ---------------- | --------------------------------------------------------- |
| `"Cat cat CAT"`       | `cat: 3`         | Case-sensitive counting would instead produce three separate one-count words: `Cat`, `cat`, `CAT`. |
| `"Ізде із ІЗДЕ"`      | `ізде: 2, із: 1` | Same as above, applied to Kazakh Cyrillic casing.          |

### 3. Stop words are not removed

No language's stop words (`the`, `и`, `және`, ...) are filtered out. The
problem statement never mentions stop-word removal, and "trending words"
boxes on real news sites routinely do show function words; inventing a
per-language stop-word list would itself be a second, unstated spec.

| Input                                   | Expected output                     | A different plausible reading                                 |
| ----------------------------------------- | ------------------------------------ | ------------------------------------------------------------------ |
| `"the cat and the dog and the bird"`     | `the: 3, and: 2, bird: 1, cat: 1, dog: 1` | Removing English stop words would drop `the` and `and` entirely. |

### 4. Inflectional forms are not normalized

Russian and Kazakh are heavily inflected: Russian "дом", "дома", "дому",
"домом" are four surface forms of one root ("house"), and Kazakh adds its
own case suffixes on top of word roots. This spec counts exact surface
forms, as they appear in the text — it does not stem or lemmatize, so those
four Russian forms count as four distinct words, each with count 1, not one
word with count 4.

This is the same reasoning as decision 3: a lemmatizer is itself a second,
much larger, per-language specification (and in practice needs a dictionary
or model beyond the standard library), so inventing one here would relocate
the ambiguity rather than resolve it. "Most common words" is read as "most
common exact spellings."

| Input                          | Expected output                                          | A different plausible reading |
| --------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| `"дом дома дому домом"`          | `дом: 1, дома: 1, домом: 1, дому: 1`                         | Lemmatizing to a shared root would instead produce `дом: 4`. |

### 5. Ranking and tie-breaking

Results are ordered by count descending; ties are broken by the casefolded
word, ascending by Unicode code point. This applies both to the visible
order of the returned list and to deciding which words make the top 10 when
more than 10 distinct words are tied at the cutoff count — the alphabetically
first 10 of the tied words are kept, the rest are dropped.

Ascending code-point order was chosen over, say, first-appearance order,
because it is independent of where in the text a word first shows up, is
trivial to implement with `sorted()`, and is the same for every reader
regardless of locale.

| Input                                                                 | Expected output                                                               | A different plausible reading |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `"zebra apple mango banana cherry date eggplant fig grape honeydew kiwi"` (each word appears once) | `apple, banana, cherry, date, eggplant, fig, grape, honeydew, kiwi, mango` (alphabetical top 10; `zebra` dropped) | First-appearance order would instead start with `zebra, apple, mango, ...` and keep `zebra`, dropping `mango` or another word instead. |

### 6. Fewer than 10 words, and no words at all

If `text` contains between 1 and 9 distinct words, all of them are returned
(no padding with `None` or zero-count placeholders). If `text` contains zero
words — including the empty string, whitespace-only text, or text made up
entirely of digits/punctuation — `top_words` returns `[]` rather than
raising an exception.

| Input           | Expected output      | A different plausible reading                          |
| ------------------ | ---------------------- | ----------------------------------------------------------- |
| `"one one two"`    | `one: 2, two: 1`       | Padding to a fixed length of 10 would instead return 8 extra `None`/`("", 0)` entries. |
| `""`                | `[]`                    | Raising `ValueError` on empty input is an equally plausible but different reading of "no input". |
| `"123 !!! ---"`     | `[]`                    | Same as above — no letters means no words, not an error.    |

### 7. Input validation

If `text` is not a `str` (e.g. `None`, `bytes`, a list of strings), `top_words`
raises `TypeError`. There is no silent coercion.

| Input   | Expected output                     |
| --------- | -------------------------------------- |
| `None`    | raises `TypeError`                    |
| `b"cat"`  | raises `TypeError`                    |
