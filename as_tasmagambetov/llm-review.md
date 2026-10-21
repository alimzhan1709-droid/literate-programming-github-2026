# LLM review

Assistant: OpenAI Codex  
Model: GPT-6.1 Sol,Max

Prompting this model with the request and specification below produced the following review. The original review was written in Russian; its English translation is recorded here.

## Prompt used

Review spec.md for completeness, consistency, and feasibility. Identify ambiguous requirements, contradictions, missing edge cases, and untestable acceptance criteria.

## Review output 

The specification still permits different implementations. The output format and sorting rules are defined, but word recognition and normalization need clarification, and several examples need correction.

### 1. The definition of a word is missing

Punctuation is described as a separator only in an example. State a general rule covering digits, apostrophes, hyphens, underscores, and whitespace.

The current specification permits several interpretations:

| Input | Possible interpretations |
|---|---|
| `"don't"` | One word, `don't`, or two words, `don` and `t` |
| `"well-known"` | One word or two words |
| `"cat2dog"` | One word, `cat2dog`; two words, `cat` and `dog`; or `catdog` after removing the digit |
| `"one_two"` | One word or two words |

Choose the intended behavior and provide exact expected outputs.

Also specify whether all Unicode letters are supported or only particular alphabets. An alphabet list ending with “etc.” does not define an exact boundary.

### 2. Case normalization is underspecified

“Case is ignored” does not identify the normalization operation.

One possible rule is:

> Each extracted word is normalized using Python's `str.lower()`. The returned words are these normalized strings.

Another possible choice is `str.casefold()`, but it can produce different results. For example, for `"Straße STRASSE"`:

- Using `lower()`: `[("strasse", 1), ("straße", 1)]`
- Using `casefold()`: `[("strasse", 2)]`

Choose an operation explicitly. Also clarify that preserving letters and diacritics still permits the specified case conversion.

Reference: [Python string case folding](https://docs.python.org/3/library/stdtypes.html#str.casefold).

### 3. Preserving diacritics does not define Unicode normalization

The same visible character can sometimes be represented by different Unicode sequences. For example, `й` can be represented as one character or as `и` followed by a combining breve.

For the Python string literal `"й и\u0306"`, specify:

- Whether both representations are counted as the same word with a count of 2.
- Whether they remain distinct.
- Whether combining marks are included in words.

NFC normalization can unify canonically equivalent representations while preserving diacritics. Explicitly state whether Unicode normalization is performed and the order of processing operations.

Reference: [Python Unicode normalization](https://docs.python.org/3/library/unicodedata.html#unicodedata.normalize).

### 4. The truncation example does not test truncation

The expression:

```python
"x " * 15 + "y"
```

contains only two unique words. Its expected output, `[("x", 15), ("y", 1)]`, tests frequency counting and returning fewer than ten items.

Correct the description of this example and remove the extra `12` before the expression.

To test the ten-word limit, use an input with more than ten unique words:

```text
Input:
"k j i h g f e d c b a"

Expected output:
[("a", 1), ("b", 1), ("c", 1), ("d", 1), ("e", 1),
 ("f", 1), ("g", 1), ("h", 1), ("i", 1), ("j", 1)]
```

This example tests tie ordering and selection at the ten-item boundary.

### 5. Three expected outputs contradict the declared output format

The Cyrillic, Kazakh, and punctuation examples omit the list brackets.

Their expected outputs should be:

| Input | Expected output |
|---|---|
| `"Привет привет ПРИВЕТ мир"` | `[("привет", 3), ("мир", 1)]` |
| `"Сәлем сәлем Қазақстан"` | `[("сәлем", 2), ("қазақстан", 1)]` |
| `"hello, hello! hello?"` | `[("hello", 3)]` |

Place `""` and `"123 !!! ---"` in separate rows so that each row represents one concrete function call.

### 6. Some specification decisions lack supporting examples

The assignment requires at least one input and expected output for each decision.

Add examples covering rules already stated in the specification:

| Input | Expected output | Rule tested |
|---|---|---|
| `"b a"` | `[("a", 1), ("b", 1)]` | Ordering when frequencies are equal |
| `"cat cats cat"` | `[("cat", 2), ("cats", 1)]` | Different word forms remain distinct |
| `"cafe café"` | `[("cafe", 1), ("café", 1)]` | Diacritics are preserved |

After clarifying word recognition and normalization, add examples for those decisions as well.

Reference: [Top words assignment](https://github.com/bjones1/literate-programming-book/blob/main/exercises/top-words.md).

### Minor corrections

- Include the original requirement to use Python 3 and place the function in `top_words.py`.
- In the Markdown file, write headings as `# Top words specification`, without wrapping the entire heading in `**`.
