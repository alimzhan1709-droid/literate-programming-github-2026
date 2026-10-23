# LLM Review of `spec.md`

Prompting Gemini 3.8 Flash, high effort, with `Review spec.md against the "Top words" prompt from the e-book to ensure there is only one plausible interpretation, checking multilingual support, tie-breaking, and edge cases` produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

---

### Review Findings:

1. **Multilingual Support (Kazakh/Russian/English):**
   - The explicit inclusion of the 9 extended Kazakh Cyrillic letters (`ә`, `ғ`, `қ`, `ң`, `ө`, `ұ`, `ү`, `һ`, `і`) prevents standard `\w+` regex pitfalls, where non-ASCII characters might be silently dropped in default ASCII modes.
   - Case folding is clearly specified for these characters.

2. **Tie-breaking Rule:**
   - Defining ascending lexicographical order by Unicode code point makes tie-breaking fully deterministic across operating systems and Python versions.

3. **Hyphen and Apostrophe Handling:**
   - Splitting on hyphens and apostrophes resolves ambiguous contractions like `"don't"` and compound words like `"step-by-step"`.
   - ~~Suggestion: You should keep apostrophes intact so words like "don't" remain single units, while splitting only hyphens.~~ (Rejected: Natural languages have differing uses of apostrophes; treating all non-alphabetic characters as uniform delimiters is simpler, more consistent, and leaves zero ambiguity).

4. **Stop Words:**
   - Explicitly stating that stop words are not removed avoids arbitrary and hidden dictionaries for English, Russian, and Kazakh.

5. **Edge Cases and Return Format:**
   - Return type (`list[tuple[str, int]]`) is strictly typed, preventing dictionary vs. list discrepancies.
   - Behavior for fewer than 10 words and empty strings is clearly mapped without unnecessary exceptions.
   - ~~Suggestion: The function should automatically read from a text file path if a string path is passed instead of raw article text.~~ (Incorrect: The assignment specifies `def top_words(text: str)`, taking an article's text directly, not a file path).