# LLM Review of spec.md

## Model and settings

- Model: ChatGPT GPT-6
- Thinking effort: Not separately specified
- Review date: October 22, 2026

## Prompt used

Review the following specification for a Python 3 function named `top_words`, which returns the 10 most frequent words in an article that may contain English, Russian, or Kazakh text.

Identify ambiguities, contradictions, incorrect expected outputs, missing edge cases, and any mismatch between the rules and examples.

Pay particular attention to:

- Unicode and multilingual word boundaries
- Unicode normalization and case-folding
- Punctuation, apostrophes, hyphens, underscores, and digits
- Frequency counting and tie-breaking
- The top-10 limit and ties at the cutoff
- Fewer than 10 distinct words and empty input
- The exact return type and output ordering
- Invalid input types
- Whether every example produces exactly its stated output

Do not silently choose behavior that the specification leaves undefined. List each problem, explain why it matters, and suggest a precise correction.

## Review output

The specification defines a clear interface, tokenization rules, normalization behavior, frequency counting, ranking, output format, and error handling. The examples cover English, Russian, and Kazakh text, as well as punctuation, digits, ties, and edge cases.

The following points were checked:

1. **Function interface:** The function accepts a string and returns a list of `(word, frequency)` tuples. The specification also defines the behavior for non-string inputs.

2. **Word boundaries:** Defining words as maximal sequences of Unicode characters whose General Category begins with `L` provides a precise tokenization rule. Punctuation, digits, whitespace, apostrophes, hyphens, and underscores separate words.

3. **Normalization and case:** NFC normalization, Unicode case-folding, and a second NFC normalization step define how equivalent spellings and different capitalization are handled.

4. **Frequency ranking:** The specification sorts by descending frequency and then ascending Unicode code-point order. This determines the order of words with equal frequencies.

5. **Top-10 selection:** The example containing `z z a b c d e f g h i j k` has 12 distinct words. The expected output contains `z` and the first nine remaining words in alphabetical order, correctly excluding `j` and `k`.

6. **Apostrophe example:** The input `cat can't cant` produces the words `cat`, `can`, `t`, and `cant`, each with frequency 1. The expected ordering is `can`, `cant`, `cat`, `t`, consistent with ascending Unicode code-point order.

7. **Hyphen example:** The input `well-known well known` produces `well` and `known`, each with frequency 2. The expected output places `known` before `well`, consistent with the tie-breaking rule.

8. **Multilingual examples:** The Russian and Kazakh examples use valid Cyrillic letters and correctly combine words that differ only in capitalization.

9. **Empty and separator-only input:** Empty text, whitespace-only text, and text containing no Unicode letters correctly produce an empty list.

10. **Remaining implementation consideration:** Unicode character classifications and case-folding behavior depend on the Unicode database bundled with Python. If identical results across different Python runtime versions are required, the implementation environment should specify a Unicode database version.

## Conclusion

The specification is sufficiently precise to guide an implementation and test it against the listed examples. No incorrect expected outputs were identified in the reviewed version.

The implementation should be checked against every example in Section 8, as required by the assignment.