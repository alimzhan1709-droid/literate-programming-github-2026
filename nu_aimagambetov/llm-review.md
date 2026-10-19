Prompting GPT-6, Instant, with "Review the following specification: ", produced
the following output; strikethrough indicates LLM output that I deemed incorrect
or irrelevant."
================================================================================

Specification review: Top words
-------------------------------

**Verdict:** The specification is clear, internally consistent, and
implementable using Python 3's standard library. All 15 examples appear
consistent with the stated rules.

There are a few details worth highlighting to avoid implementation mistakes.

1\. Tokenization and validation are separate steps
--------------------------------------------------

The specification requires two distinct operations:

1. Split the input into maximal consecutive sequences of allowed Latin or
   Cyrillic characters.
2. Discard any resulting token containing characters from both alphabets.

For example:

```python
"helloпривет hello привет"
```

The first token, `helloпривет`, must be **discarded entirely**, not split into
`hello` and `привет`.

The resulting frequencies are:

```python
OrderedDict({"hello": 1, "привет": 1})
```

Kazakh and Russian characters belong to the same Cyrillic set, so `сәлемпривет`
is a valid single word.

## <s>2\. Unicode handling needs to be explicit</s>

<s>The specification correctly avoids Unicode normalization and restricts which
characters are permitted.</s>

<s>For example:</s>

<s> </s><s>`ё` is one allowed character.`е\u0308` consists of an allowed `е`
followed by a disallowed combining mark.</s>

* <s>`İ` is not an allowed Latin character and therefore acts as a
  delimiter.</s> <s> Avoid using `str.isalpha()`, `str.islower()`, or `\w` for
  tokenization because they accept characters outside the specified
  alphabets.</s>

3\. Sorting is deterministic
----------------------------

The sorting rule:

```python
(-frequency, word)
```

ensures that words are ordered by descending frequency, with alphabetical order
breaking ties.

The result must contain at most 10 entries, even when more than 10 distinct
valid words occur.

4\. Reference implementation
----------------------------

Here is an implementation that follows the specification.

```python
import re
from collections import OrderedDict

LATIN = (
    "abcdefghijklmnopqrstuvwxyz"
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
)

CYRILLIC = (
    "абвгдежзийклмнопрстуфхцчшщъыьэюяё"
    "АБВГДЕЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯЁ"
    "әғқңөұүһіӘҒҚҢӨҰҮҺІ"
)

ALLOWED = LATIN + CYRILLIC
TOKEN_PATTERN = re.compile(f"[{ALLOWED}]+")

LATIN_SET = set(LATIN)
CYRILLIC_SET = set(CYRILLIC)


def top_words(text: str) -> OrderedDict:
    frequencies = {}

    for token in TOKEN_PATTERN.findall(text):
        word = token.lower()

        if not (
            all(char in LATIN_SET for char in word)
            or all(char in CYRILLIC_SET for char in word)
        ):
            continue

        frequencies[word] = frequencies.get(word, 0) + 1

    sorted_words = sorted(
        frequencies.items(),
        key=lambda item: (-item[1], item[0])
    )

    return OrderedDict(sorted_words[:10])
```

5\. Suggested specification improvements
----------------------------------------

| Area               | Assessment           | Recommendation                                           |
| ------------------ | -------------------- | -------------------------------------------------------- |
| Character sets     | Clear                | Keep explicit character lists                            |
| Tokenization       | Clear                | Emphasize maximal consecutive sequences                  |
| Mixed alphabets    | Clear                | Keep the discard-entire-token rule                       |
| Case conversion    | Clear                | Specify validation after `str.lower()` as already done   |
| Sorting            | Clear                | No changes needed                                        |
| Return type        | Clear                | Optionally annotate `OrderedDict[str, int]`              |
| <s>Performance</s> | <s>Not specified</s> | <s>Optionally define expectations for large articles</s> |
| Unicode            | Clear                | Keep the no-normalization rule                           |

<s>One subtle consideration: the specification validates tokens **after**
lowercasing. That ordering should remain explicit, since Python's Unicode
lowercase behavior can sometimes change the number of code points.</s>

6\. Complexity
--------------

Let $n$ be the number of characters in the input and $k$ the number of distinct
valid words.

* Tokenization and frequency counting: $O(n)$
* Sorting: $O(k \log k)$
* Total time: $O(n + k \log k)$
* Auxiliary space: $O(n + k)$ in this implementation, accounting for the token
  list returned by `findall()`.

For typical news articles, this is efficient.

---

**Final assessment:** The specification is ready for implementation. I would not
change its functional requirements. The most important tests are mixed-script
tokens, combining Unicode characters, equal-frequency ordering, and the 10-word
limit. <s> One useful addition would be a test containing mixed-case Kazakh
Cyrillic characters to verify that case folding and frequency counting work
correctly across all supported alphabets.</s>
