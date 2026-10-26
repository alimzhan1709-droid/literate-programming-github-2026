# Specification: Trending Words

## Setup
- LLM: LLM: OpenAI GPT-6

## 1. Purpose

A news website displays a "Trending words" box beside each article. The function `top_words` finds the 10 most common words in an article and returns each word with the number of times it occurs. Articles may be written in English, Russian, or Kazakh.

## 2. Function interface

- Language: Python 3.9 or later.
- Use only the Python standard library.
- File: `top_words.py`.
- Function signature: `def top_words(text: str) -> list[tuple[str, int]]`.
- The function takes exactly one argument: the article text.
- The function does not print output, read files, access the network, or modify global state.
- If `text` is not an instance of `str`, the function raises `TypeError`. The exception message is not specified.
- A subclass of `str` is accepted. Convert it to an exact `str` before processing.
- Every returned word must be an exact `str`.

## 3. Return value

The function returns a list of `(word, count)` tuples.

- `word` is the normalized form of a word, as defined in Section 5.
- `count` is the number of occurrences of that normalized word in the article.
- Each distinct normalized word appears at most once.
- Each count is a positive integer.
- The list contains no more than 10 items.
- If the article contains fewer than 10 distinct words, return all of them.
- If the article contains no words, return `[]`.
- The result must always be a list, including when it is empty or has one item.

## 4. What counts as a word

Apply the following rules from left to right:

1. Normalize the complete input text to Unicode NFC using `unicodedata.normalize("NFC", text)` before tokenization.
2. A letter is any character for which Python's `str.isalpha()` returns `True`.
3. A combining mark is a character whose Unicode category is `Mn` or `Mc`, except for variation selectors. Variation selectors are U+180B–U+180D, U+180F, U+FE00–U+FE0F, and U+E0100–U+E01EF.
4. A word starts with a letter.
5. After a word starts, it may continue with any letter or combining mark.
6. An apostrophe (`'`, U+0027, or `’`, U+2019) or a hyphen-minus (`-`, U+002D) may be part of a word only when it has a letter immediately before it and a letter immediately after it. The joiner is retained in the word.
7. A joiner at the beginning or end of a word separates words. Two joiners in a row also separate words.
8. A combining mark is included only when it directly follows a letter or another combining mark already in the word. A mark after a digit, joiner, or other separator is not part of a word.
9. Every other character separates words. This includes whitespace, digits, underscores, punctuation, symbols, emoji, variation selectors, enclosing marks (Unicode category `Me`), and format characters (Unicode category `Cf`).
10. Digits never belong to words. For example, `abc123def` produces `abc` and `def`, while `2024` produces no word.
11. Use the same rules for English, Russian, Kazakh, and other scripts. Do not detect the language or apply language-specific tokenization.

Examples of valid letters include Latin letters, Cyrillic letters, and Kazakh letters such as `ә`, `ғ`, `қ`, `ң`, `ө`, `ұ`, `ү`, `і`, and `һ`.

## 5. Word normalization

After tokenizing the text, normalize each word using these steps in order:

1. Replace every curly apostrophe `’` (U+2019) with the straight apostrophe `'` (U+0027).
2. Convert the complete word to lowercase using Python's `str.lower()`.

Do not apply Unicode normalization again after lowercasing.

Additional requirements:

- Words that are equal after these steps are counted together.
- Do not remove accents or combining marks.
- Do not treat `ё` as equivalent to `е`, or `й` as equivalent to `и`.
- Do not stem or lemmatize words. For example, `cat` and `cats` are different words.
- Do not remove stop words. Words such as `the`, `и`, and `және` count like any other words.
- Latin `i` and Cyrillic `і` are different characters and remain different words.
- Do not use `str.casefold()`; use `str.lower()` exactly as specified.

## 6. Counting

- Count every occurrence of every token after normalization.
- Repeated occurrences contribute one to the count each time.
- Tokens with the same normalized form share one count.
- Words with different normalized forms must be counted separately, even if their meanings are similar.

## 7. Sorting and selecting the top 10

Sort distinct normalized words by these keys, in order:

1. Higher frequency first.
2. If frequencies are equal, the word that comes first in ascending Unicode code-point order, as determined by Python string comparison using `<`.

Equivalently, the complete list is sorted by `(-count, word)`. Return only the first 10 items after sorting. The tie-break rule also determines which words are selected when more than 10 distinct words have the same frequency.

## 8. Examples and expected outputs

Each example shows the input text and the exact expected Python return value.

| Input | Expected output | Rule tested |
|---|---|---|
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | Count occurrences and sort by frequency |
| `"Cat cat CAT"` | `[("cat", 3)]` | Case-insensitive counting |
| `"Алматы алматы АЛМАТЫ"` | `[("алматы", 3)]` | Lowercase Cyrillic words |
| `"Қазақ қазақ ҚАЗАҚ"` | `[("қазақ", 3)]` | Lowercase Kazakh letters |
| `"b a c a b"` | `[("a", 2), ("b", 2), ("c", 1)]` | Unicode code-point tie-break |
| `"cat, cat. cat!"` | `[("cat", 3)]` | Punctuation separates words |
| `"cat_dog"` | `[("cat", 1), ("dog", 1)]` | Underscore separates words |
| `"abc123def 2024"` | `[("abc", 1), ("def", 1)]` | Digits separate words and are not words |
| `"don't don’t"` | `[("don't", 2)]` | Curly and straight apostrophes are equivalent |
| `"well-known well-known"` | `[("well-known", 2)]` | Internal hyphen-minus joins a word |
| `"'cat' dogs'"` | `[("cat", 1), ("dogs", 1)]` | Apostrophes at word edges are separators |
| `"a--b"` | `[("a", 1), ("b", 1)]` | Consecutive joiners separate words |
| `"Ёж ёж"` | `[("ёж", 2)]` | Cyrillic lowercase |
| `"еж ёж"` | `[("еж", 1), ("ёж", 1)]` | `е` and `ё` are distinct |
| `"и\u0306 й"` | `[("й", 2)]` | NFC normalization before tokenization |
| `"i і"` | `[("i", 1), ("і", 1)]` | Latin and Cyrillic characters remain distinct |
| `"a b c d e f g h i j k l"` | `[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]` | Maximum of 10 items and tie-breaking |
| `"cat dog"` | `[("cat", 1), ("dog", 1)]` | Return all words when fewer than 10 exist |
| `""` | `[]` | Empty input |
| `"   \t\n "` | `[]` | Whitespace-only input |
| `"123 ! @#$"` | `[]` | Input containing no words |

### Realistic article example

Input:

```text
Москва — столица России. Москва большая, и в Москве много людей. Астана — столица Казахстана.
```

Expected output:

```python
[
    ("москва", 2),
    ("столица", 2),
    ("астана", 1),
    ("большая", 1),
    ("в", 1),
    ("и", 1),
    ("казахстана", 1),
    ("людей", 1),
    ("много", 1),
    ("москве", 1),
]
```

The article contains 11 distinct normalized words. Only 10 are returned. The word `россии` is excluded because, among the words tied with a count of 1, it comes after the selected words in Unicode code-point order. `москва` and `москве` are separate words.

## 9. Performance

The expected running time is `O(n + d log d)`, where `n` is the number of characters in the input and `d` is the number of distinct normalized words. This bound ignores the cost of comparing words. Any implementation that meets the specified behavior and expected complexity is acceptable.

## 10. Conformance

An implementation conforms to this specification only if it follows all stated rules for tokenization, normalization, counting, sorting, top-10 selection, return types, and error handling.

Every example in Section 8 must produce exactly the specified output.
