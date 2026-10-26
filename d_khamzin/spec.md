# Top words specification

## Function
`top_words(text: str) -> list[tuple[str, int]]` in `top_words.py`.
Python 3.9 or newer. The whole text may be processed in memory. 

## Processing order
1. Type check (Decision 9).
2. NFC-normalize the text.
3. Replace `’` (U+2019) with `'` (U+0027).
4. Apply `str.casefold()` (locale-independent), then NFC-normalize again.
5. Tokenize into words (Decisions 1-2).
6. Count words, sort (Decision 7), and keep at most 10 (Decision 6).

## Decisions

1. **Letters.** A letter is any character for which `str.isalpha()` is true
   (any script: Latin, Cyrillic including Kazakh ә і ң ғ ү ұ қ ө һ, Greek,
   CJK, ...), or whose Unicode category starts with `M` (combining marks, e.g.
   the stress mark U+0301). Every other character (digits, punctuation,
   whitespace, symbols, format characters such as the soft hyphen U+00AD, and
   quote-like characters such as `‘` U+2018 and `` ` ``) is a separator.
   `ʼ` (U+02BC) is a letter under `isalpha()`, so it stays inside a word and
   is not the same as `'`.
2. **Word.** A word is a maximal run of letters in which exactly one connector
   (`'` U+0027 or `-` U+002D) may appear between two letters; the connector
   stays inside the word. Two or more connectors in a row, a leading or
   trailing connector, or any other dash (en dash, em dash, U+2010) split
   words. A hyphen followed by a line break splits words. A run must contain
   at least one character for which `isalpha()` is true; a run made only of
   combining marks is discarded.
3. **Apostrophes.** Because of step 3 of the processing order, `don't` and
   `don’t` are the same word.
4. **Case and normalization.** Output uses the casefolded, NFC form
   (`Straße` becomes `strasse`; Greek `ς` becomes `σ`; `İ` becomes `i` followed
   by U+0307). `ё` and `е` are distinct. Cyrillic `і` (U+0456) and Latin `i`
   are distinct.
5. **No filtering, no segmentation.** No stemming, no stop-word removal; "the"
   counts. Only English, Russian, and Kazakh are guaranteed; other scripts
   follow the same rules, and languages written without spaces (e.g. Chinese)
   are not segmented, so an unbroken run of such letters is one word.
6. **Result size.** At most 10 entries; fewer if fewer distinct words exist.
7. **Ordering.** Descending count; ties broken by ascending Unicode code point
   order of the word (Python's default string comparison).
8. **Empty input.** Empty text, or text with no words, returns `[]`.
9. **Invalid input.** Any argument that is not an instance of `str` (`int`,
   `bool`, `None`, `bytes`, ...) raises `TypeError`. Subclasses of `str` are
   accepted. The string `"123"` is valid and returns `[]`.

## Examples

Inputs are Python literals.

| Input | Expected output | Decision |
| --- | --- | --- |
| `"cat dog cat bird dog cat"` | `[("cat",3),("dog",2),("bird",1)]` | basic |
| `"b a"` | `[("a",1),("b",1)]` | 7 |
| `"a1b 22 c"` | `[("a",1),("b",1),("c",1)]` | 1 |
| `"日本 日本"` | `[("日本",2)]` | 1, 5 |
| `"\u0443\u0434\u0430\u0301\u0440\u0435\u043d\u0438\u0435"` (stress mark) | the same string, count 1 | 1 |
| `"x \u0301 y"` (lone combining mark) | `[("x",1),("y",1)]` | 2 |
| `"well-known well known"` | `[("known",1),("well",1),("well-known",1)]` | 2, 7 |
| `"rock'n'roll"` | `[("rock'n'roll",1)]` | 2 |
| `"don''t a--b"` | `[("a",1),("b",1),("don",1),("t",1)]` | 2 |
| `"'cat' cat"` | `[("cat",2)]` | 2 |
| `"dogs'"` | `[("dogs",1)]` | 2 |
| `"word\u2014word"` (em dash) | `[("word",2)]` | 2 |
| `"a\u2010b"` (U+2010) | `[("a",1),("b",1)]` | 2 |
| `"a\u00adb"` (soft hyphen) | `[("a",1),("b",1)]` | 1 |
| `"hyphen-\nated"` | `[("ated",1),("hyphen",1)]` | 2 |
| `"don't don\u2019t"` | `[("don't",2)]` | 3 |
| `"don\u2018t"` (U+2018) | `[("don",1),("t",1)]` | 1 |
| `"don\u02bct don't"` (U+02BC) | `[("don't",1),("don\u02bct",1)]` | 1, 7 |
| `"Cat cat CAT"` | `[("cat",3)]` | 4 |
| `"Stra\u00dfe STRASSE"` | `[("strasse",2)]` | 4 |
| `"\u03bf\u03b4\u03bf\u03c2 \u039f\u0394\u039f\u03a3"` (οδος ΟΔΟΣ) | `[("\u03bf\u03b4\u03bf\u03c3",2)]` | 4 |
| `"\u0130"` (İ) | `[("i\u0307",1)]` | 4 |
| `"\u0451\u0436 \u0435\u0436"` (ёж еж) | `[("\u0435\u0436",1),("\u0451\u0436",1)]` | 4, 7 |
| `"\u0451\u0436 \u0435\u0308\u0436"` (ё precomposed vs. decomposed) | `[("\u0451\u0436",2)]` | 4 |
| `"\u0456 i"` (Cyrillic і, Latin i) | `[("i",1),("\u0456",1)]` | 4, 7 |
| `"Қазақстан қазақ ҚАЗАҚ"` | `[("қазақ",2),("қазақстан",1)]` | 4 |
| `"яблоко apple"` | `[("apple",1),("яблоко",1)]` | 7 |
| `"l k j i h g f e d c b a"` | `[("a",1),("b",1),("c",1),("d",1),("e",1),("f",1),("g",1),("h",1),("i",1),("j",1)]` | 6, 7 |
| `""` and `"... !!!"` | `[]` | 8 |
| `"123"` | `[]` | 9 |
| `123`, `True`, `None`, `b"cat"` | `TypeError` | 9 |