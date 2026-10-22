# Specification: Top Words

## 1. Purpose

The function `top_words` finds the 10 most frequent distinct words in an article and returns each word together with its frequency.

## 2. Function interface

- Language: Python 3.
- Function name: `top_words`.
- Input: one Python string containing the article text.
- Return type: a list of tuples. Each tuple contains a word as a string and its frequency as a positive integer.
- The function does not modify the input string.

The function returns at most 10 tuples.

## 3. Definition of a word

A word is a maximal, non-empty sequence of Unicode letters.

A Unicode letter is a character whose Unicode General Category begins with `L`. This includes uppercase and lowercase Latin letters, Cyrillic letters, and the additional Cyrillic letters used in Kazakh.

For example, `a`, `Я`, `ә`, `қ`, `ң`, `ө`, `ұ`, `ү`, `і`, `ғ`, and `һ` are valid letters.

Every character that is not a Unicode letter separates words. In particular:

- Whitespace separates words.
- Punctuation separates words.
- Digits separate words and are not included in words.
- Apostrophes separate words.
- Hyphens and dashes separate words.
- Underscores separate words.
- Any other character that is not a Unicode letter separates words.

A string containing no Unicode letters produces no words.

## 4. Unicode normalization and case

Before tokenization, the function performs these operations in order:

1. Normalize the input to Unicode NFC.
2. Apply Unicode `casefold()` to make matching case-insensitive.
3. Normalize the result to Unicode NFC again.

Words that become identical after these operations are counted as the same word.

For example, `Apple`, `APPLE`, and `apple` are all counted as `apple`.

The word returned in the result is the normalized, case-folded form, not the original spelling.

## 5. Counting

Each occurrence of a word contributes exactly one to that word's frequency.

Repeated occurrences are counted independently. Words are compared after normalization and case-folding.

## 6. Ranking and selection

The function ranks words using the following rules:

1. Words with higher frequencies appear first.
2. If two words have the same frequency, the word that comes first in ascending Unicode code-point order appears first.
3. After sorting all distinct words, the function returns only the first 10.

The tie-breaking rule applies before the limit of 10 is applied. It therefore determines which words are included when multiple words are tied for the final available position.

## 7. Output

The return value is a list of `(word, frequency)` tuples.

- Each word appears at most once.
- Frequencies are positive integers.
- Results are sorted according to Section 6.
- If the input contains fewer than 10 distinct words, all distinct words are returned.
- If the input contains no words, the function returns an empty list: `[]`.
- The function returns a list even when it contains zero or one tuple.

## 8. Examples and expected outputs

All expected outputs below use the required list-of-tuples format.

| Input | Expected output | Rule tested |
|---|---|---|
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | Frequency counting |
| `"Apple apple APPLE"` | `[("apple", 3)]` | Case-insensitive matching |
| `"cat, dog! cat."` | `[("cat", 2), ("dog", 1)]` | Punctuation separates words |
| `"cat can't cant"` | `[("can", 1), ("cant", 1), ("cat", 1), ("t", 1)]` | Apostrophes separate words; ties use Unicode code-point order |
| `"well-known well known"` | `[("known", 2), ("well", 2)]` | Hyphens separate words; ties use Unicode code-point order |
| `"cat123dog cat 123 dog"` | `[("cat", 2), ("dog", 2)]` | Digits separate words |
| `"cat_dog cat dog"` | `[("cat", 2), ("dog", 2)]` | Underscores separate words |
| `"Ана ана Бала"` | `[("ана", 2), ("бала", 1)]` | Cyrillic letters and case-folding |
| `"Қазақ қазақ Тіл"` | `[("қазақ", 2), ("тіл", 1)]` | Kazakh letters and case-folding |
| `"b a c"` | `[("a", 1), ("b", 1), ("c", 1)]` | Alphabetical ordering when frequencies tie |
| `"z z a b c d e f g h i j k"` | `[("z", 2), ("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1), ("f", 1), ("g", 1), ("h", 1), ("i", 1)]` | Top-10 limit; `j` and `k` are excluded |
| `"cat dog"` | `[("cat", 1), ("dog", 1)]` | Fewer than 10 distinct words |
| `""` | `[]` | Empty input |
| `"   \t\n "` | `[]` | Whitespace-only input |
| `"123 ! @#$"` | `[]` | Input with no Unicode letters |

## 9. Preconditions and errors

The input must be a Python string. If the argument is not a string, the function raises `TypeError`.

No other input restrictions apply. An empty string and a string containing only separators are valid inputs and return an empty list.

## 10. Conformance requirement

An implementation conforms to this specification only if it follows the stated normalization, tokenization, counting, ranking, selection, output-format, and error-handling rules.

Each example in Section 8 must produce exactly the specified output.