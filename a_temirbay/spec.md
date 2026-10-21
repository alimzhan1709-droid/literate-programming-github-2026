Specification: top_words
========================

Purpose:
--------
top_words returns the up to n most frequent words in a piece of plain text, 
along with their counts. Supports English, Russian, and Kazakh text (or a mix). 
The function is language-agnostic and does not use language-specific rules.

Interface:
----------
File: top_words.py, Python 3.
Signature:
    def top_words(text: str, n: int = 10) -> list[tuple[str, int]]:

Rules & Validation:
-------------------
1. Parameter text: Must be a str. If not (including None), raises TypeError. Checked first.
2. Parameter n:
   - Defaults to 10. Must be a non-negative strict integer.
   - Type validation: If n is not an int or is a bool (e.g., True/False, floats), raises TypeError. 
     This check runs before negativity check (e.g., n=-1.5 raises TypeError).
   - Value validation: If n is a negative integer, raises ValueError.
   - n = 0 returns [].
3. Return Value: Returns up to n pairs of (word, count), ordered by frequency 
   (descending), then by Unicode codepoint order (ascending) for ties.
4. Empty or Non-Word Text: Empty strings "", whitespace-only strings, or text containing 
   only digits/punctuation return [].
5. Side Effects: Must not modify its input and must not print anything.

Implementation Details & Edge Cases:
------------------------------------
1. Unicode Normalization: Input is normalized to NFC upfront.
2. Apostrophe Mapping: Typographic variants (‘, ’, ʼ, ʻ, `) are mapped to 
   standard straight apostrophes (') via str.maketrans before matching.
3. Word Definition & Boundaries:
   - Letter Runs: Maximal runs of Unicode letters (including combining marks U+0300–U+036F).
   - Internal Joiners & Compounds: Single internal apostrophes (') or hyphens (-) join letter runs. 
     Chained compounds (e.g., `state-of-the-art`, `don't`) are treated as single compound tokens.
   - Leading/Trailing Punctuation: Leading or trailing hyphens and apostrophes act as boundaries 
     and are stripped (e.g., `-word` yields `word`).
   - Multiple Consecutive Punctuation: Multiple consecutive hyphens or apostrophes (e.g., `co--op`, `don''t`) 
     act as separators, splitting the words (yielding `co` and `op`).
   - Separators: Digits, underscores, whitespace, punctuation, and format characters (category Cf like U+200B) 
     act as word separators. Pure-digit tokens are ignored.
4. Case Conversion: Tokens are lowercased using standard Python str.lower(), followed by an NFC re-normalization step.
5. Letter ё and е: Kept distinct; ties sorted by Unicode codepoint order.
6. Plain Text Only: HTML, Markdown, and URLs are not stripped.