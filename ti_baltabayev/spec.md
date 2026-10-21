# Specification: `top_words`

## Purpose

A news site shows a "Trending words" box beside each article. `top_words` finds the 10 most common words in an article's text and how many times each appears. Articles may be in English, Russian, or Kazakh (or a mix).

## Interface

```python
def top_words(text: str) -> list[tuple[str, int]]
```

- Python 3.9 or newer, standard library only (`unicodedata` is allowed). The function lives in `top_words.py`.
- Returns a list of `(word, count)` tuples, at most 10 long, ordered by count descending, then by word ascending (see Rule 7).

### Notation in the tables

- `cat: 3, dog: 2` is shorthand for `[('cat', 3), ('dog', 2)]`, and `(empty)` means `[]`.
- `\uXXXX` inside an input or output denotes that single code point. Every invisible, combining, or look-alike character is written this way, so that the tables are unambiguous.
- Where an expected output contains a character that is easy to confuse, its code point is given in parentheses.

## Algorithm (normative)

1. If `text` is not an instance of `str`, raise `TypeError` (Rule 11).
2. Normalize to Unicode NFC (not NFKC).
3. Delete every U+0301 (combining acute accent, used as a Russian stress mark) and every U+00AD (soft hyphen).
4. Replace U+2019 (’) and U+02BC (ʼ) with U+0027 (').
5. Convert to lowercase with `str.lower()`. Do not use `casefold()`, and do not re-normalize afterwards.
6. Extract words (Rule 2 and Rule 3).
7. Count occurrences of each word.
8. Sort by `(-count, word)` (Rule 7).
9. Return the first 10 entries (Rule 8).

Each rule below states one decision, followed by examples whose output would differ under some other plausible reading.

---

## Rule 1: Input and output types

The input is plain text in one `str`. The output is a list of `(str, int)` tuples. An empty input, or an input with no words, returns `[]`. If fewer than 10 distinct words exist, all of them are returned. There is no input size limit; the reference implementation runs in time linear in the text length (plus sorting the distinct words).

| Input | Expected output | Rejected alternative |
|---|---|---|
| `cat dog cat bird dog cat` | cat: 3, dog: 2, bird: 1 | sorted by first occurrence or alphabetically |
| `cat dog` | cat: 1, dog: 1 | padding to 10 entries |
| `` (empty string) | (empty) | raising an error |
| `123 !!! ...` | (empty) | digits or punctuation returned as words |

## Rule 2: What a word is

Two character classes are used, both evaluated on the text after steps 2-5:

- A **letter** is any character `c` for which `c.isalpha()` is `True` (this covers Latin, Cyrillic, and the Kazakh letters Ә Ғ Қ Ң Ө Ұ Ү Һ І).
- A **mark** is any character whose Unicode general category starts with `M` (`unicodedata.category(c)[0] == "M"`).

A **run** is one letter followed by any number of letters and marks. A mark that does not follow a letter or another mark of the same run (for example at the start of the text or after a space) is a separator. Everything else that is not part of a run also separates words: whitespace (including `\n`, `\t`, U+00A0), punctuation, digits, underscore, and format characters such as U+200B, U+200C, and U+200D.

Marks stay attached because NFC cannot compose every letter-plus-mark sequence (for example, `q` + U+0323 has no precomposed form), and a mark is not alphabetic by itself. Without this rule such a word would be split in two.

A word is a run, optionally joined to further runs as described in Rule 3.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `Hello, world! Hello.` | hello: 2, world: 1 | punctuation kept: `hello,` |
| `2024 cat 2024` | cat: 1 | numbers counted as words |
| `abc123def` | abc: 1, def: 1 | one word `abc123def` |
| `snake_case` | case: 1, snake: 1 | one word `snake_case` |
| `кошка \u2014 собака, кошка` | кошка: 2, собака: 1 | dash counted as a word |
| `cat\ndog\u00a0cat` | cat: 2, dog: 1 | NBSP/newline glued into words |
| `cat\u200bdog` (zero-width space) | cat: 1, dog: 1 | one word `cat\u200bdog` |
| `a a b` | a: 2, b: 1 | single-letter words dropped |
| `q\u0323a q\u0323a` (q + dot below, no precomposed form) | `q\u0323a`: 2 | word split into `q: 2, a: 2` |
| `\u0323cat cat` (leading mark that survives step 3) | cat: 2 | mark kept as part of the word |
| `\u0301cat cat` (U+0301 is deleted in step 3 before extraction) | cat: 2 | |

## Rule 3: Internal apostrophes and hyphens

A single `'` (U+0027) or `-` (U+002D) is part of the word only if it is **immediately preceded by a letter or mark belonging to the current word (that is, by the end of a run, including a run already joined to earlier runs) and immediately followed by a letter**. A word may therefore contain several joins (`a-b-c`, `rock'n'roll`). Otherwise it is a separator: at the start or end of a run, doubled, or next to any other non-letter, including another apostrophe or hyphen. Only ASCII hyphen-minus counts as a hyphen. U+2010, U+2011, en dash, and em dash are separators. (The typographic apostrophes ’ and ʼ have already become `'` in step 4.)

| Input | Expected output | Rejected alternative |
|---|---|---|
| `well-known well-known` | well-known: 2 | split into `well`, `known` |
| `из-за дождя из-за` | из-за: 2, дождя: 1 | split into `из`, `за` |
| `don't don\u2019t` | don't: 2 | `don`, `t`; or `\u2019` kept as a different character |
| `'cat' cat` | cat: 2 | quotes kept: `'cat'` |
| `dogs' dogs` | dogs: 2 | trailing apostrophe kept |
| `-cat- cat` | cat: 2 | hyphens kept |
| `cat - dog` | cat: 1, dog: 1 | `-` as a word |
| `cat--dog` | cat: 1, dog: 1 | one word `cat--dog` |
| `cat-'dog` | cat: 1, dog: 1 | one word `cat-'dog` |
| `cat'-dog` | cat: 1, dog: 1 | one word `cat'-dog` |
| `a-1 1-a` | a: 2 | `a-` or `-a` kept |
| `cat\u2014dog` (em dash) | cat: 1, dog: 1 | one word |
| `cat\u2010dog` (U+2010 hyphen) | cat: 1, dog: 1 | one word `cat\u2010dog` |
| `cat\u2011dog` (non-breaking hyphen) | cat: 1, dog: 1 | one word `cat\u2011dog` |
| `rock'n'roll` | rock'n'roll: 1 | split at each apostrophe |
| `a-b-c a-b-c` | a-b-c: 2 | only the first join kept |
| `cat-\u0323dog` (hyphen followed by a mark, not a letter) | cat: 1, dog: 1 | one word `cat-\u0323dog` |
| `q\u0323'a q\u0323'a` (run ending in a mark, then `'`) | `q\u0323'a`: 2 | split at the apostrophe |

## Rule 4: Case-insensitive; output is lowercase

Words are compared and returned in lowercase, using `str.lower()` (not `casefold()`), including Cyrillic and Kazakh letters. Lowercasing never merges different letters such as `ß` and `ss`. Lowercasing can produce a combining mark (`İ` lowercases to `i` + U+0307); that mark stays attached under Rule 2, and the result is a different word from plain `istanbul`.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `Cat CAT cat` | cat: 3 | three separate entries |
| `Москва МОСКВА` | москва: 2 | |
| `Қазақстан ҚАЗАҚСТАН қазақстан` | қазақстан: 3 | Kazakh letters not folded |
| `Ілияс ілияс` | ілияс: 2 | Kazakh І not folded |
| `Straße STRASSE` | strasse: 1, straße: 1 | `casefold()` merging them: `strasse: 2` |
| `\u0130stanbul istanbul` | istanbul: 1, `i\u0307stanbul`: 1 | `İ` splitting the word into `i` and `stanbul`; or merged into `istanbul: 2` |

## Rule 5: Unicode normalization

Canonically equivalent spellings count as the same word, because the text is normalized to NFC (step 2). Compatibility equivalents (full-width letters, ligatures) are **not** unified, because NFKC is not used. The typographic apostrophes U+2019 and U+02BC become `'`. U+02BB (ʻ, as in Uzbek `oʻz`) is **not** mapped: it is a modifier letter, so it is part of the word and `oʻz` differs from `o'z`. U+2018 and the backtick are separators. The Russian stress mark U+0301 is dropped, so a stressed word equals its unstressed form. The soft hyphen U+00AD is dropped, so a word hyphenated for display equals the plain word. Because NFC runs before step 3, Latin `é` written as `e` + U+0301 first becomes the single character `é` and keeps its accent. U+0301 on a letter that has no precomposed form with it (for example `q` + U+0301) is simply dropped.

**Known exception for Cyrillic.** NFC also composes `\u0433` + U+0301 into `\u0453` (ѓ) and `\u043a` + U+0301 into `\u045c` (ќ), so these two stressed letters keep their accent and do not equal the unstressed word. This is accepted behavior, not a bug: the stress mark is only dropped when NFC leaves it as a separate character. In Russian it affects only a stressed `г` or `к`, which is very rare.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `cafe\u0301 caf\u00e9` (decomposed, precomposed) | café: 2 | two different entries |
| `\u0438\u0306 \u0439` (decomposed й, precomposed й) | `\u0439` (precomposed й): 2 | two entries or the marks split words |
| `молоко\u0301 молоко` (stressed, unstressed) | молоко: 2 | `моло` + `ко`, or two entries |
| `\u0433\u0301 \u0433` (г + stress mark, plain г) | `\u0433`: 1, `\u0453` (ѓ): 1 | merged: `\u0433: 2` |
| `cat-\u00addog` (soft hyphen deleted, then `-` joins) | cat-dog: 1 | `cat`, `dog` |
| `cat'\u0301dog` (stress mark deleted, then `'` joins) | cat'dog: 1 | `cat`, `dog` |
| `cat\u200bdog cat-\u00addog` (U+200B separates, U+00AD is deleted) | cat: 1, cat-dog: 1, dog: 1 | both treated alike |
| `o\u02bcz o'z` | o'z: 2 | two entries |
| `o\u02bbz o'z` (U+02BB is a letter, not mapped) | o'z: 1, `o\u02bbz`: 1 | merged: `o'z: 2` |
| `inter\u00adnational international` (soft hyphen) | international: 2 | `inter` + `national`, or two entries |
| `\uff43\uff41\uff54 cat` (full-width `ｃａｔ`) | cat: 1, `\uff43\uff41\uff54`: 1 | merged: `cat: 2` |

## Rule 6: No linguistic processing

There are no stop-word lists, no stemming or lemmatization, no minimum word length, and no merging of look-alike letters.

This is intentional: for ordinary articles the result will be dominated by function words (`the`, `of`, `и`, `в`, `не`). Filtering them, if the product wants it, is the caller's job and is outside this function. Note that a caller who filters the result afterwards gets fewer than 10 useful words, because the cutoff is applied inside this function. If the product needs filtered top-10 lists, the interface must change (for example an `exclude` or `n` parameter) and this spec must be revised; that is a product decision and is out of scope here.

Two consequences product owners should know: possessives (`putin's`) are counted separately from the base word, and text typed with Latin look-alike letters inside Cyrillic or Kazakh words (a Latin `a` or `i` for `а` or `і`) is counted as different words.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `the cat the dog the` | the: 3, cat: 1, dog: 1 | stop words removed |
| `cat cats` | cat: 1, cats: 1 | merged: `cat: 2` |
| `все всё все` (ё is U+0451) | все: 2, всё: 1 | `ё` folded into `е` |
| `a \u0430` (Latin a U+0061, Cyrillic а U+0430) | a: 1, `\u0430` (Cyrillic а): 1 | look-alike letters merged |
| `\u043a\u0061\u0442 \u043a\u0430\u0442` (first word has Latin a inside Cyrillic letters) | `\u043a\u0061\u0442`: 1, `\u043a\u0430\u0442`: 1 | mixed-script word repaired or merged |
| `Putin's Putin` | putin: 1, putin's: 1 | possessive `'s` stripped |

## Rule 7: Ordering and ties

Sort by count descending. Words with equal counts are sorted ascending by their lowercase form using Python's default string comparison (Unicode code point order). Consequences:

- Latin sorts before Cyrillic.
- Kazakh letters absent from the basic Russian alphabet (e.g. ә U+04D9) sort after `я`; `ё` (U+0451) sorts after `я` (U+044F).
- `-` (U+002D) sorts before every letter, so `a-b` sorts before `ab`.
- A non-ASCII Latin letter such as `é` (U+00E9) sorts after every ASCII letter at the position where two words first differ, so `zebra` sorts before `élan`.
- A word sorts before any longer word that begins with it.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `pear apple pear apple fig` | apple: 2, pear: 2, fig: 1 | first-occurrence order for ties |
| `zebra Apple` | apple: 1, zebra: 1 | first occurrence, or case-sensitive order |
| `hello привет Hello Привет` | hello: 2, привет: 2 | Cyrillic before Latin |
| `я ә а` | а: 1, я: 1, ә: 1 | locale-aware (Kazakh) alphabetical order |
| `ё я` | я: 1, ё: 1 | `ё` placed after `е` (Russian alphabet order) |
| `a-b ab` | a-b: 1, ab: 1 | `ab` first |
| `élan zebra` | zebra: 1, élan: 1 | accents ignored when sorting (`élan` first) |

## Rule 8: Limit of 10 and ties at the boundary

Exactly the first 10 entries of the sorted list are returned, so ties at the 10th position are cut by Rule 7.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `l k j i h g f e d c b a` | a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1, j: 1 | all 12 returned (ties included) |
| `z z z a b c d e f g h i j` | z: 3, a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1 | `j` included, or 11 entries |

## Rule 9: Mixed languages and other scripts

There is no language detection. Any mix of English, Russian, and Kazakh in one text is handled by the same rules.

Other scripts are not supported, but the behavior is deterministic: the same rules apply. In particular, scripts written without spaces (Chinese, Japanese) have `isalpha()` true for their characters, so an unbroken stretch of them is a single "word".

| Input | Expected output | Rejected alternative |
|---|---|---|
| `Привет привет hello Hello сәлем` | hello: 2, привет: 2, сәлем: 1 | per-language lists |
| `қазақ ҚАЗАҚ ән` | қазақ: 2, ән: 1 | |
| `\u4f60\u597d\u4e16\u754c` (four CJK characters) | the four-character string: 1 | word segmentation (out of scope) |

## Rule 10: No markup or URL handling

The input is treated as plain text. HTML tags, HTML entities, URLs, and e-mail addresses are not recognized; their letter runs are ordinary words.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `visit example.com or example.com` | com: 2, example: 2, or: 1, visit: 1 | URL removed or kept as one token |
| `<b>cat</b>` | b: 2, cat: 1 | tags stripped |
| `&amp; &nbsp;` | amp: 1, nbsp: 1 | entities decoded to `&` and a space |

## Rule 11: Invalid input

Any argument that is not an instance of `str` (including `None` and `bytes`) raises `TypeError`. Subclasses of `str` are accepted and processed like any other `str`.

| Input | Expected output | Rejected alternative |
|---|---|---|
| `None` | `TypeError` | returns `[]` |
| `b"cat"` | `TypeError` | decoded as UTF-8 |
| `42` | `TypeError` | converted with `str()` |
| `class S(str): ...` then `S("cat cat")` | cat: 2 | `TypeError` for subclasses |

---

## Appendix: reference implementation (non-normative)

This implementation passes every example in this document. It is provided because the obvious regex shortcut (`[^\W\d_]+(?:['-][^\W\d_]+)*`) is **not** equivalent to the rules above: `\w` and `\d` disagree with `isalpha()` for characters such as `²`, `½` and `Ⅷ`, and a regex has no direct way to express Rule 2's mark handling. If a regex is used anyway, it must be checked against every row of the tables.

```python
import unicodedata
from collections import Counter


def top_words(text: str) -> list[tuple[str, int]]:
    if not isinstance(text, str):
        raise TypeError("text must be a str")
    s = unicodedata.normalize("NFC", text)
    s = s.replace("\u0301", "").replace("\u00ad", "")
    s = s.replace("\u2019", "'").replace("\u02bc", "'")
    s = s.lower()

    words = []
    cur = []
    i, n = 0, len(s)
    while i < n:
        if not s[i].isalpha():
            i += 1
            continue
        # a letter, then any letters and combining marks
        cur.append(s[i])
        i += 1
        while i < n and (s[i].isalpha() or unicodedata.category(s[i]).startswith("M")):
            cur.append(s[i])
            i += 1
        # a single ' or - directly between two letters joins runs
        if i + 1 < n and s[i] in "'-" and s[i + 1].isalpha():
            cur.append(s[i])
            i += 1
            continue
        words.append("".join(cur))
        cur = []

    counts = Counter(words)
    return sorted(counts.items(), key=lambda kv: (-kv[1], kv[0]))[:10]
```