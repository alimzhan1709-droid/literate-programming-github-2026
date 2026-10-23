Prompting Gemini 3.5 Flash-Lite, medium effort, with `Review spec.md` produced
the following output; strikethrough indicates LLM output that I deemed incorrect
or irrelevant.

Here is the Python 3 implementation for the `top_words` function. It uses
Python's built-in re module with full Unicode support to correctly handle
English, Russian, and Kazakh characters (including specific letters like ә, ғ,
қ, ң, ө, ұ, ү, і, һ and Cyrillic characters), and collections.Counter for
efficient frequency counting.

`top_words.py`

```python
import re
from collections import Counter
from typing import List, Tuple


def top_words(text: str) -> List[Tuple[str, int]]:
  """Finds the 10 most common words in an article's text.

  Supports English, Russian, and Kazakh languages.

  Args:
      text (str): The article text.

  Returns:
      List[Tuple[str, int]]: A list of up to 10 tuples containing the word
      and its frequency, ordered from most common to least common.
  """
  if not text:
    return []

  # Convert text to lowercase and extract words using Unicode-aware \w+ regex.
  # In Python 3, \w automatically matches English, Russian, and Kazakh characters.
  words = re.findall(r"\w+", text.lower(), re.UNICODE)

  # Count word frequencies
  word_counts = Counter(words)

  # Return the top 10 most common words
  return word_counts.most_common(10)
```

**How It Works**

* Unicode Word Matching (`\w`): Python 3's regular expressions treat \\w as
  Unicode-aware by default. This ensures that words containing Russian Cyrillic
  letters (e.g., привет, мир) and Kazakh specific letters (e.g., әлем,
  қазақстан, <s>ұ행</s>) are tokenized correctly rather than being split or
  ignored.
* Case Insensitivity:`.lower()` ensures that words starting sentences (e.g.,
  "Cat" vs "cat") are correctly grouped together.
* Efficiency: `collections.Counter.most_common(10)` runs in <em>O(N log K)</em>
  time (where *N* is the number of words and *K*\=10), making it highly
  efficient for processing large news articles.

**Example Usage**

```
# English Example
text_en = "cat dog cat bird dog cat"
print(top_words(text_en))
# Output: [('cat', 3), ('dog', 2), ('bird', 1)]

# Kazakh / Russian Mixed Example
text_multilang = "Қазақстан қазақстан мен мир мир привет мир"
print(top_words(text_multilang))
# Output: [('мир', 3), ('қазақстан', 2), ('мен', 1), ('привет', 1)]
```