# `top_words` Specification

## 1. Overview

`top_words(text: str) -> list[tuple[str, int]]` returns up to 10 most frequent words and their occurrence counts.

The function processes English, Russian, Kazakh, and mixed-language text. The required processing order is defined across §§2–6.

Results are deterministic for identical inputs under the same Python and Unicode database versions.

## 2. Input Validation

- Accept `str` and its subclasses.
- Raise `TypeError` for non-string inputs.
- Raise `ValueError` if the raw input contains any lone surrogate code points (U+D800–U+DFFF).
- Perform the type check before the surrogate check. Scan the raw input for surrogates before normalization.
- Obtain the plain string using `str.__str__(text)`, without invoking overridden subclass methods.
- Use unbound `str` methods, such as `str.casefold(value)`, for subsequent string operations where applicable.
- Return only plain `str` objects in the result, never instances of a `str` subclass.
- Do not impose undocumented input-size limits or produce external side effects.

Validation must precede all text processing.

## 3. Normalization

After validation, normalize the input in this order:

1. Apply Unicode NFC normalization.
2. Apply Python's `str.casefold()`.
3. Apply Unicode NFC normalization again.

Use the Unicode database provided by Python. Results may differ between Unicode database versions.

The normalization procedure is applied to stopword entries as well. Each entry in the stopword lists must already be stored in the final normalized, case-folded form.

## 4. Tokenization

A letter is any Unicode character whose General Category starts with `L`. A mark is any Unicode character whose General Category is `M*` (`Mn`, `Mc`, or `Me`).

A letter group follows `L(?:M*L)*M*`: one or more letters, each optionally followed by marks. Marks cannot start a letter group. An unattached mark acts as a boundary and is discarded. In particular, a mark immediately after a separator is unattached; in `a-\u0301b`, the tokens are `a` and `b`.

The only permitted internal separators are:

- ASCII hyphen-minus: `-` (U+002D)
- Hyphen: `‐` (U+2010)
- ASCII apostrophe: `'` (U+0027)
- Right single quotation mark: `’` (U+2019)

A token follows `LETTER_GROUP(?:SEPARATOR LETTER_GROUP)*`, where `SEPARATOR` is exactly one of the four permitted characters above. A separator not participating in this grammar acts as a boundary and is discarded. This rule also discards leading and trailing separators and makes consecutive separators terminate the token; separators are never combined.

Digits, whitespace, punctuation, symbols, and control or format characters that are not permitted separators act as boundaries. Leading and trailing separators are discarded. Consecutive separators terminate the current token; they are not combined.

Do not unify separator variants. For example, `don't` and `don’t`, or `e-mail` and `e‐mail`, are distinct tokens. Stopword matching uses the exact normalized token spelling, including separator characters; therefore, a stopword containing ASCII apostrophe U+0027 does not match the corresponding token with U+2019. No stopword entry below currently contains an apostrophe; if such an entry is added, its listed apostrophe must be U+0027, with U+2019 remaining distinct.

Preserve each token's normalized, case-folded spelling.

Examples:

| Input | Tokens |
|---|---|
| `state-of-the-art` | `state-of-the-art` |
| `hello--world` | `hello`, `world` |
| `can'’t` | `can`, `t` |
| `don't don’t` | `don't`, `don’t` |
| `e-mail e‐mail` | `e-mail`, `e‐mail` |
| `abc123def` | `abc`, `def` |
| `hello_world` | `hello`, `world` |
| `café` | `café` |
| `a-\u0301b` | `a`, `b` |
| `canʼt` | `canʼt` |
| `a\u00adb` | `a`, `b` |
| `a\u200cb a\u200db` | `a`, `b`, `a`, `b` |
| `e‑mail` | `e`, `mail` |
| `a−b` | `a`, `b` |
| `你好世界` | `你好世界` |

Here, `ʼ` is U+02BC, `­` is U+00AD, `‌` is U+200C, `‍` is U+200D, `‑` is U+2011, and `−` is U+2212.

Text written in scripts that normally do not separate words with spaces, such as Chinese, Japanese, and Thai, is not segmented linguistically. A continuous run of letters is treated as one token unless a character defined above creates a boundary.

This is an expected limitation of the specified tokenizer, not a request for language-specific segmentation. Other lookalikes are governed only by their Unicode General Category: U+02BC `ʼ` is a letter and stays within a token; U+00AD, U+200C, and U+200D are format characters and create boundaries; U+2011 and U+2212 create boundaries rather than acting as permitted separators.

## 5. Stopword Filtering

Remove tokens that exactly match an entry in the fixed, combined English, Russian, and Kazakh stopword lists below. All entries are stored in their normalized, case-folded form, using NFC normalization, Python case folding, and NFC normalization again, as specified in §3.

**English stopwords:**

```python
ENGLISH_STOPWORDS = {
    "a", "an", "and", "are", "as", "at", "be", "been", "being",
    "but", "by", "can", "could", "did", "do", "does", "doing",
    "for", "from", "had", "has", "have", "having", "he", "her",
    "here", "hers", "him", "his", "how", "i", "if", "in", "into",
    "is", "it", "its", "me", "more", "most", "my", "no", "nor",
    "not", "of", "on", "once", "only", "or", "other", "our",
    "ours", "out", "over", "own", "same", "she", "should", "so",
    "some", "such", "than", "that", "the", "their", "theirs",
    "them", "then", "there", "these", "they", "this", "those",
    "through", "to", "too", "under", "until", "up", "very", "was",
    "we", "were", "what", "when", "where", "which", "while", "who",
    "whom", "why", "will", "with", "would", "you", "your", "yours"
}
```

**Russian stopwords:**

```python
RUSSIAN_STOPWORDS = {
    "а", "без", "более", "бы", "был", "была", "были", "было",
    "быть", "в", "вам", "вас", "весь", "во", "вот", "все",
    "всё", "всего", "всех", "вы", "где", "да", "даже", "для",
    "до", "его", "ее", "её", "если", "есть", "ещё", "же",
    "за", "здесь", "и", "из", "или", "им", "их", "к", "как",
    "ко", "когда", "кто", "ли", "либо", "мне", "может", "мы",
    "на", "над", "надо", "наш", "не", "него", "нее", "неё",
    "нет", "ни", "них", "но", "ну", "о", "об", "однако",
    "он", "она", "они", "оно", "от", "по", "под", "при",
    "про", "с", "сам", "свою", "себе", "себя", "так", "также",
    "такой", "там", "те", "тем", "то", "того", "тоже", "той",
    "только", "том", "ты", "у", "уже", "чего", "чей", "чем",
    "что", "чтобы", "эта", "эти", "это", "я"
}
```

**Kazakh stopwords:**

```python
KAZAKH_STOPWORDS = {
    "ал", "ала", "алдында", "арқылы", "әр", "әрбір", "әркім",
    "әрқашан", "әртүрлі", "әрине", "әлі", "әдетте", "ба", "бе",
    "бен", "бері", "бірақ", "бір", "бірге", "бірде", "біреу",
    "бірнеше", "бірінші", "бірқатар", "бірқалыпты", "біріншіден",
    "бол", "болады", "болған", "болса", "болып", "болу", "бұл",
    "бұған", "бұдан", "бұрын", "бүкіл", "да", "де", "демек",
    "дейін", "деген", "дегенмен", "дәл", "екен", "емес", "енді",
    "үшін", "ғана", "ғой", "және", "жоқ", "жөн", "жоғары",
    "кейін", "кейбір", "керек", "кезде", "кім", "кімге", "кімнің",
    "көп", "ма", "ме", "мен", "мұнда", "мұны", "мұның", "мына",
    "мынадай", "мынау", "мың", "не", "немесе", "нәрсе", "осы",
    "осыған", "осындай", "осында", "осыдан", "осымен", "осының",
    "оған", "одан", "ол", "олар", "олардың", "оларға", "оларды",
    "онда", "ондай", "оның", "оң", "өте", "өз", "өзі",
    "өзін", "өзінің", "өзге", "өздері", "сайын", "себебі",
    "сияқты", "сол", "сонда", "сондай", "соң", "сонымен",
    "сондықтан", "тағы", "тек", "туралы", "ты", "у", "үш",
    "яғни", "и"
}
```

The combined stopword set is the union of these three sets. Every listed entry is already in the §3 final normalized form (NFC, Python casefold, then NFC). Matching is exact and language-independent. No language detection, stemming, lemmatization, substring matching, or additional stopword removal is permitted.

Separator characters are not interchangeable. For example, `don't` and `don’t` are distinct entries. A token is filtered only if its exact normalized spelling appears in the combined set.

## 6. Counting and Sorting

Count all occurrences of tokens remaining after stopword filtering.

Sort the distinct tokens by:

1. Frequency, descending.
2. Token string, ascending Unicode code-point order.

Use ordinary Python string ordering, not locale-aware sorting.

Return the first 10 `(word, count)` tuples. Return fewer than 10 if fewer than 10 distinct tokens remain; an empty valid input therefore returns `[]`. Ties at the tenth position are resolved by the same token-string ordering; do not return extra tokens to include all tied entries.

## 7. Reference Tests

The following examples define the expected behavior. Stopword-related outputs are determined by the fixed lists in §5.

```python
top_words("Cat dog cat. Собака кошка собака. Мысық ит мысық.")
# [('cat', 2), ('мысық', 2), ('собака', 2),
#  ('dog', 1), ('ит', 1), ('кошка', 1)]

top_words("The quick brown fox jumps over the lazy dog.")
# [('brown', 1), ('dog', 1), ('fox', 1), ('jumps', 1),
#  ('lazy', 1), ('quick', 1)]

top_words("state-of-the-art AI model's performance")
# [('ai', 1), ("model's", 1), ('performance', 1),
#  ('state-of-the-art', 1)]

top_words("Straße STRASSE")
# [('strasse', 2)]

top_words("café cafe\u0301")
# [('café', 2)]

top_words("hello--world abc123def")
# [('abc', 1), ('def', 1), ('hello', 1), ('world', 1)]

top_words("banana apple cherry banana apple cherry")
# [('apple', 2), ('banana', 2), ('cherry', 2)]

top_words("")
# []

top_words("the and of мы и")
# []

top_words(123)
# Raises TypeError

top_words("\ud800")
# Raises ValueError
```

Additional required tests:

```python
class CustomStr(str):
    def __str__(self):
        raise AssertionError("overridden __str__ must not be called")

    def casefold(self):
        raise AssertionError("overridden casefold must not be called")

result = top_words(CustomStr("Cat cat"))
assert result == [("cat", 2)]
assert type(result[0][0]) is str
```

```python
top_words("a\u0301 á")
# [('á', 2)]

top_words("a-\u0301b")
# [('a', 1), ('b', 1)]

top_words("don't don’t e-mail e‐mail")
# [('don't', 1), ('don’t', 1),
#  ('e-mail', 1), ('e‐mail', 1)]
```

The following test verifies Unicode code-point ordering for equal frequencies:

```python
top_words("z é a")
# [('a', 1), ('z', 1), ('é', 1)]
```

The following test verifies the 10-token limit and tie-breaking at the cutoff:

```python
top_words("k j i h g f e d c b a")
# [('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1),
#  ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]
```

This test has more than 10 distinct tokens with unequal frequencies:

```python
top_words("a a a b b c d e f g h i j k")
# [('a', 3), ('b', 2), ('c', 1), ('d', 1), ('e', 1),
#  ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]
```

The preceding 11-token test also makes the equal-frequency tie cross the tenth-place cutoff: `k` is excluded by code-point ordering.

The following cases cover letters (`L`), each kind of mark (`Mn`, `Mc`, `Me`), numbers (`N`), punctuation (`P`), symbols (`S`), separators (`Z`), control characters (`Cc`), and format characters (`Cf`). Marks attach only after letters; the remaining shown categories create boundaries.

```python
top_words("a\u0301b a\u0903b a\u20ddb")
# [('aःb', 1), ('a⃝b', 1), ('áb', 1)]

top_words("a1b a_b a+b a.b a b a\u200bb a\u0001b")
# [('a', 7), ('b', 7)]
```

The following tests verify that distinct separator variants remain distinct:

```python
top_words("don't don’t")
# [("don't", 1), ("don’t", 1)]

top_words("e-mail e‐mail")
# [('e-mail', 1), ('e‐mail', 1)]
```

The subclass test above verifies overridden `__str__` and `casefold` methods. Also verify raw-surrogate validation on a subclass:

```python
class SurrogateStr(str):
    def __str__(self):
        raise AssertionError("overridden __str__ must not be called")

top_words(SurrogateStr("a\ud800"))
# Raises ValueError
```

Stopword-related expected outputs must agree with the three fixed lists in §5.

## 8. Implementation Contract

The implementation must follow this specification exactly, use no locale-dependent behavior, have no external side effects, and return no more than 10 results. For input length `n` and `u` distinct retained tokens, tokenization and counting should take O(n) time; ranking may use a full sort in O(u log u) time or a top-10 selection strategy. Space usage is O(u), excluding the input and returned result. No undocumented input-size limit is permitted.

Identical inputs must produce identical results under the same Python and Unicode database versions.