# Top words specification

A news site shows a "Trending words" box beside each article. `top_words` finds the 10 most common words of an article and how often each appears. Articles may be in English, Russian or Kazakh (Cyrillic or Latin letters; no language detection is done).

## Interface

- Python 3, file `top_words.py`, function `top_words(text: str) -> list[tuple[str, int]]`.
- Returns at most 10 `(word, count)` tuples, ordered as described in "Output order".
- If `text` is not an instance of `str` (for example `None` or `bytes`), raise `TypeError`.

## Processing steps

Apply these steps in this order:

1. **Normalize** `text` to Unicode NFC (`unicodedata.normalize("NFC", text)`).
2. **Tokenize** into words (next section).
3. **Normalize apostrophes:** replace every apostrophe inside a word (U+2019 or U+02BC) with U+0027.
4. **Case-fold** each word with `str.casefold()`, then normalize the result to NFC again. Words are compared and returned in this form. (`casefold` can change a word: `ß` becomes `ss`, and `İ` (U+0130) becomes `i` + U+0307.)
5. **Count** identical words and **sort** them.

## Definition of a word

- A *word character* is any character whose Unicode general category starts with `L` (letter) or `M` (combining mark), **except** U+02BC, which is an apostrophe (below). Letters of any script count, including Latin, Cyrillic and the Kazakh letters ә ғ қ ң ө ұ ү һ і.
- A *connector* is an apostrophe (U+0027, U+2019 or U+02BC) or a hyphen-minus (U+002D). No other character is a connector. In particular U+2010 and U+2011 hyphens, U+2018, U+00B4, the backtick and the soft hyphen U+00AD are *not* connectors.
- A *word* is a maximal sequence of word characters in which any two neighbouring word characters may be joined by exactly **one** connector. A connector is part of the word only when it has a word character immediately before **and** immediately after it. So `rock'n'roll`, `well-known` and `don't-do` are single words, while `'dog'`, `dogs'`, `-dog`, `well--known` and `a'-b` are not: there the connector is a separator.
- A word must contain at least one letter (category `L*`). A sequence made only of combining marks is discarded.
- Every other character is a **separator**: whitespace (including newlines and tabs), punctuation, symbols, em/en dashes, underscore, and all digits. Digits are never part of a word and numbers are never counted: `b2b` gives `b` and `b`; `2024` gives nothing.
- No stemming, lemmatization, script splitting or unification: `привет-world` is one word; Latin `a` (U+0061) and Cyrillic `а` (U+0430) are different letters; `ё` and `е` are different letters.
- Scripts written without spaces (for example Chinese) get no special handling: a run of letters is one word.
- No stop words are removed: `the`, `и` and `және` are counted like any other word.

## Output order

- Sort by count descending; ties are broken by the word ascending in Unicode code point order (Python's default string comparison). A word that is a prefix of another sorts first (`well` before `well-known`).
- Return the first 10 items of that order. If there are fewer than 10 distinct words, return all of them. If there are none, return `[]`.
- The result is a new `list` of `tuple`s `(str, int)`.

## Examples

Each row exercises a decision above: under another plausible reading of the original problem the output would differ. `\uXXXX` in an input is a single character with that code point.

| Decision | Input | Expected output |
| --- | --- | --- |
| Basic counting | `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` |
| Case-insensitive (Latin) | `"Cat CAT cat"` | `[("cat", 3)]` |
| Case-insensitive (Cyrillic) | `"Привет привет ПРИВЕТ"` | `[("привет", 3)]` |
| Kazakh letters | `"Қазақстан қазақстан Әлем"` | `[("қазақстан", 2), ("әлем", 1)]` |
| ё and е are different; ties by code point (е U+0435 < ё U+0451) | `"еж ёж"` | `[("еж", 1), ("ёж", 1)]` |
| Punctuation separates | `"hi, hi! hi? (hi)"` | `[("hi", 4)]` |
| Whitespace of any kind separates | `"cat\ncat\tdog"` | `[("cat", 2), ("dog", 1)]` |
| Apostrophes U+0027, U+2019, U+02BC are one word, written as U+0027 | `"don't don\u2019t don\u02BCt"` | `[("don't", 3)]` |
| Edge apostrophes are separators | `"'dog' dog"` | `[("dog", 2)]` |
| Chained apostrophes | `"rock'n'roll"` | `[("rock'n'roll", 1)]` |
| Internal hyphen joins; ties by code point (prefix sorts first) | `"well-known well known"` | `[("known", 1), ("well", 1), ("well-known", 1)]` |
| Doubled hyphen separates | `"well--known"` | `[("known", 1), ("well", 1)]` |
| Apostrophe and hyphen together in one word | `"don't-do"` | `[("don't-do", 1)]` |
| Apostrophe next to hyphen separates | `"a'-b"` | `[("a", 1), ("b", 1)]` |
| Em dash is a separator | `"a \u2014 b"` | `[("a", 1), ("b", 1)]` |
| U+2010 hyphen is a separator | `"a\u2010b"` | `[("a", 1), ("b", 1)]` |
| Connector next to a digit is an edge | `"a-1 3-d"` | `[("a", 1), ("d", 1)]` |
| Digits are separators | `"2024 abc 123abc"` | `[("abc", 2)]` |
| Letters and digits mixed | `"b2b"` | `[("b", 2)]` |
| Underscore separates | `"foo_bar"` | `[("bar", 1), ("foo", 1)]` |
| No stop words removed | `"the the a"` | `[("the", 2), ("a", 1)]` |
| Tie-break alphabetical | `"b a c b a"` | `[("a", 2), ("b", 2), ("c", 1)]` |
| casefold, not lower (ß → ss) | `"Straße STRASSE"` | `[("strasse", 2)]` |
| casefold result is re-normalized (İ → i + U+0307) | `"\u0130"` | `[("i\u0307", 1)]` |
| No script splitting | `"привет-world приветworld"` | `[("привет-world", 1), ("приветworld", 1)]` |
| Latin a (U+0061) and Cyrillic а (U+0430) are different | `"a \u0430"` | `[("a", 1), ("\u0430", 1)]` |
| Latin i (U+0069) and Kazakh і (U+0456) are different | `"i \u0406"` | `[("i", 1), ("\u0456", 1)]` |
| NFC normalization (и + U+0306 vs й U+0439) | `"и\u0306 й"` | `[("й", 2)]` |
| Combining marks only is not a word | `"\u0301"` | `[]` |
| Ten words, all ties: first ten alphabetically | `"a b c d e f g h i j k"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` |
| Cut-off after ten with a higher count first | `"z z a b c d e f g h i j"` | `[("z", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` |
| Soft hyphen U+00AD is a separator | `"co\u00ADoperate"` | `[("co", 1), ("operate", 1)]` |
| U+2018 is a separator; U+2019 at the edge too | `"\u2018dog\u2019"` | `[("dog", 1)]` |
| Scripts without spaces: a run of letters is one word | `"中文"` | `[("中文", 1)]` |
| Empty text | `""` | `[]` |
| No words | `"123 !!!"` | `[]` |
| Not a str: None | `None` | raises `TypeError` |
| Not a str: bytes | `b'abc'` | raises `TypeError` |