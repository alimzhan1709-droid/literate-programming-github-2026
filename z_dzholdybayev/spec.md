
# Specification: `top_words`

## 1. Overview and Interface

Write a Python 3 function named `top_words` in a module named `top_words.py`.

```python
def top_words(text: str) -> list[tuple[str, int]]:
    """Return the top 10 most frequent words in `text` and their counts."""

Return Format

The function returns a list of (word, count) tuples:

  - word (str): Lowercased word string.
  - count (int): Number of occurrences in the text.
  - The list contains at most 10 elements, sorted by:
    1.  count in descending order.
    2.  word in ascending lexicographical (Unicode code point) order for ties.

2. Specification Decisions and Disambiguation

Below are the explicit architectural decisions resolving all ambiguities of the
original problem, accompanied by test examples that disambiguate alternative
plausible interpretations.

Decision 1: Multilingual Word Definition (English, Russian, Kazakh)

  - Rule: A word is a contiguous sequence of alphabetic characters belonging to:
      - Basic Latin (English: a-z, A-Z).
      - Cyrillic (Russian and Kazakh: а-я, А-Я, ё, Ё).
      - Extended Kazakh Cyrillic characters: ә, ғ, қ, ң, ө, ұ, ү, һ, і (and
        their uppercase forms Ә, Ғ, Қ, Ң, Ө, Ұ, Ү, Һ, І).
  - Digits (0-9), punctuation, symbols, and whitespace are delimiters and not
    part of any word.
  - Alternative rejected: Treating digits as parts of words (e.g., "covid19").
    Under this spec, "covid19" becomes the word "covid".

| Input                             |  Expected Output                          | Plausible Alternative Output  |
| :-------------------------- | :---------------------------------------------- | :---------------------- |
| `"Сәлем, әлем! Привет мир. Hello world 2026."` | `[('hello', 1), ('world', 1), ('мир', 1), ('привет', 1), ('сәлем', 1), ('әлем', 1)]` | Includes numbers: `[('2026', 1), ...]` or breaks Kazakh `ә`: `[('лем', 1), ...]` |

Decision 2: Punctuation, Hyphens, and Apostrophes

  - Rule: Hyphens (-), apostrophes (', ’), and underscores (_) are treated
    strictly as delimiters, separating words.
  - Words separated by hyphens (e.g., compound words) or apostrophes (e.g.,
    English contractions) are split into distinct tokens.
  - Alternative rejected: Preserving internal hyphens or apostrophes (e.g.,
    keeping "don't" intact or "state-of-the-art" as one token).

| Input                  | Expected Output                                  | Plausible Alternative Output          |
| :--------------------- | :----------------------------------------------- | :------------------------------------ |
| `"don't step-by-step"` | `[('step', 2), ('by', 1), ('don', 1), ('t', 1)]` | `[("don't", 1), ("step-by-step", 1)]` |

Decision 3: Case Folding and Normalization

  - Rule: Case folding is applied using Python’s .casefold() (or .lower())
    before counting. All output words are lowercase.
  - Kazakh/Russian Cyrillic uppercase letters map strictly to their lowercase
    equivalents (e.g., "І" -> "і", "Қ" -> "қ").
  - Alternative rejected: Case-sensitive counting where "The" and "the" are
    treated as two distinct words.

| Input                        | Expected Output              | Plausible Alternative Output                                   |
| :--------------------------- | :--------------------------- | :-------------------------------------------------- |
| `"Алма алма АЛМА Ізде іЗдЕ"` | `[('алма', 3), ('ізде', 2)]` | Case-sensitive: `[('алма', 1), ('Алма', 1), ('АЛМА', 1), ...]` |

Decision 4: Deterministic Tie-Breaking

  - Rule: When two or more words have identical frequencies, ties are broken
    lexicographically in ascending order by Unicode code point.
  - Alternative rejected: Preserving first-appearance order or non-deterministic
    dictionary order.

| Input                             | Expected Output                       | Plausible Alternative Output |
| :--------------------------- | :----------------------------------------- | :------------------------------ |
| `"banana cherry apple date"` | `[('apple', 1), ('banana', 1), ('cherry', 1), ('date', 1)]` | First-appearance order: `[('banana', 1), ('cherry', 1), ('apple', 1), ('date', 1)]` |

Decision 5: Stop Words

  - Rule: No stop-word filtering is applied. All valid words, regardless of
    frequency or grammatical role (e.g., English "the", Russian "и", Kazakh
    "және"), are counted equally.
  - Alternative rejected: Stripping common grammatical particles.

| Input                     | Expected Output               | Plausible Alternative Output                           |
| :------------ | :------------------------------------ | :----------------------------------- |
| `"the cat and the dog және мысық"` | `[('the', 2), ('and', 1), ('cat', 1), ('dog', 1), ('мысық', 1), ('және', 1)]` | Filtering stop words: `[('cat', 1), ('dog', 1), ('мысық', 1)]` |

Decision 6: Fewer than 10 Unique Words and Empty Input

  - Rule: If the text contains fewer than 10 unique words, the function returns
    all unique words found. It does not pad the list with None or raise an
    exception.
  - If text contains no alphabetic words (or is empty), the function returns an
    empty list [].
  - Alternative rejected: Raising ValueError on empty input or padding the list
    to length 10.

| Input                     | Expected Output            | Plausible Alternative Output                       |
| :------------------------ | :------------------------- | :------------------------------------------------- |
| `"one one two"`           | `[('one', 2), ('two', 1)]` | Padded list: `[('one', 2), ('two', 1), None, ...]` |
| `""` or `"1234 !!! ??? "` | `[]`                       | Raises `ValueError("No words found")`              |

Decision 7: Input Validation

  - Rule: If text is not an instance of str, the function raises a
    TypeError("Input must be a string").


