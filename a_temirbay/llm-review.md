# LLM Review for `spec.md`

## Review Metadata
- **LLM Used:** Gemini 3
- **Prompt Used:** `review this spec` followed by the full text of `spec.md`.

---

## LLM Review Output

This is an exceptionally thorough, well-thought-out, and rigorous specification. It anticipates many common edge cases (such as the bool subclassing issue in Python where isinstance(True, int) is True, Unicode normalization order, apostrophe normalization, and tie-breaking rules).

Here is a review of the specification, highlighting its strengths, a couple of minor implementation considerations, and suggestions for ensuring smooth execution.

### Strengths of the Specification
1. Strict Type & Value Validation for n: Explicitly blocking bool types (since bool is a subclass of int in Python) and defining the exact order of checks (TypeError before negativity/ValueError) prevents subtle bugs.
2. Robust Multilingual & Unicode Handling:
  - Upfront NFC normalization ensures consistency.
  - Treating ё and е as distinct aligns with standard Russian/Kazakh lexicographical expectations while keeping things deterministic via Unicode codepoint sorting for ties.
  - Explicitly mapping typographic apostrophe variants to a standard straight apostrophe handles copy-pasted text cleanly.
3. Clear Boundary & Compound Rules: Distinguishing between single internal joiners (state-of-the-art, don't) and multiple consecutive punctuation (co--op) provides unambiguous behavior for tokenization.

### Implementation Considerations & Tips
~~1. Handling bool for n in Python~~
~~Since isinstance(True, int) evaluates to True in Python, your validation check for n must explicitly check for bool first:~~
~~```python~~
~~if isinstance(n, bool) or not isinstance(n, int):~~
~~    raise TypeError("n must be an integer")~~
~~if n < 0:~~
~~    raise ValueError("n must be non-negative")~~
~~```~~
~~The spec already correctly implies this, but it's worth keeping front-of-mind during coding.~~~~

~~2. Unicode Letter Runs and Python's re Module~~
~~The spec defines letter runs as maximal runs of Unicode letters including combining marks.~~
~~- Python's standard built-in re module does not support full Unicode property escapes (like \p{L}) by default unless using the third-party regex library or carefully crafting character classes.~~
~~- Alternatively, you can iterate through characters using unicodedata.category(c) (checking if it starts with 'L' or is in ('Mn', 'Mc', 'Me') for combining marks) or use the third-party regex module (regex.findall(r'\p{L}+', text)). If you want to keep dependencies strictly standard-library-only, you'll want to specify how letter categories are identified.~~~~

~~3. Pure-Digit and Separator Logic~~
~~The spec states: "Pure-digit tokens are ignored. Digits, underscores, whitespace, punctuation, and format characters... act as word separators."~~
~~Ensure that internal digits aren't accidentally swallowed if a word contains numbers (e.g., web2.0 — dots are punctuation, so it would split into web and 0 unless handled specifically, which the spec handles correctly by treating punctuation as separators).~~

### Verdict
Ready for implementation. The specification is clear, edge-case-resistant, and leaves very little room for ambiguous interpretation. You can proceed directly to writing top_words.py and its corresponding unit tests based on this document.