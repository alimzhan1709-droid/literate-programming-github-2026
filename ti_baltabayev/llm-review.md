# LLM Review of `spec.md`

## Review metadata

- **Prompt used:** `Review spec.md`
- **Model:** OpenAI GPT-6
- **Thinking effort:** Not available in the conversation metadata; not asserted.
- **Source reviewed:** `spec.md`
- **Annotation convention:** Strikethrough indicates LLM output deemed incorrect or irrelevant by the author. No strikethrough annotations have been added because the author has not identified specific review statements to reject.

## LLM output

# Review `spec.md`

I've reviewed the specification for `top_words`, a Python function that returns the 10 most frequent words in an article, with support for English, Russian, Kazakh, and mixed-language text.

**Overall assessment: the spec is unusually thorough and precise.** It defines not just ordinary word counting, but Unicode normalization, combining marks, apostrophes, hyphens, sorting, edge cases, and invalid inputs. It also includes a reference implementation and concrete examples for testing.

My main concern is not the overall design, but a few places where the normative rules and the reference implementation may not agree.

## 1. Key findings

- **Core algorithm is sound**

  The reference implementation follows the intended normalization, tokenization, counting, sorting, and top-10 selection sequence. I found no clear contradiction between the core algorithm and the normative rules.

- **Unicode version is not pinned**

  Python 3.9+ is specified, but Unicode character properties and normalization data can vary between Python versions. This may affect edge cases involving less common characters.

- **The product definition is intentionally narrow**

  Stop words, stemming, lemmatization, URL parsing, and markup removal are explicitly excluded. These are not implementation bugs, but the product team should understand the consequences.

## 2. What is particularly well specified

- **Unicode processing order:** NFC normalization, removal of specific characters, apostrophe mapping, and lowercase conversion happen in a defined sequence. This prevents many implementation differences.
- **Word boundaries:** Letters, combining marks, digits, underscores, and internal apostrophes and hyphens are handled explicitly.
- **Deterministic output:** Sorting uses `(-count, word)`, with Python's Unicode string comparison for ties.
- **Boundary behavior:** The specification explains exactly which words survive when multiple words tie for tenth place.
- **Input validation:** Non-string inputs raise `TypeError`, including `None`, bytes, and numbers.

These details make the specification suitable for implementation and automated testing.

## 3. What I would improve

| Priority | Recommendation | Why |
|---|---|---|
| Medium | Document Unicode-version dependence | Clarifies reproducibility across Python releases. |
| Medium | Add automated tests for every example | Prevents regressions in tricky Unicode cases. |
| Low | Add a concise acceptance checklist | Makes the spec easier for a developer or reviewer to validate. |
| Low | Keep the reference implementation synchronized with the rules | Prevents future changes from making the examples and code disagree. |

### One important product consideration

The function returns the most frequent words *before* any downstream filtering. For example, common words such as `the`, `и`, and `в` can occupy most of the ten slots. Filtering those words afterward may leave fewer than ten useful terms.

The specification explicitly acknowledges this limitation and correctly treats changing the interface to support exclusions as a separate product decision, rather than silently adding stop-word filtering.

## 4. Final verdict

**8.5/10 — Strong specification**

It is detailed, testable, and unusually careful about Unicode edge cases. I would approve it for implementation, with one recommendation: add a test suite covering all examples in the document and decide whether exact cross-version Unicode reproducibility matters.

**If this is for a coding assessment or an AI-assisted implementation task**, the spec is already strong enough to serve as the authoritative contract. The next useful step would be to implement `top_words.py` and run a test suite against every normative example, rather than rewriting the specification.

---

## Follow-up changes made to `spec.md`

After this review, the following changes were reported as made:

- Clarified that Unicode behavior can depend on the Python runtime's Unicode database.
- Added an acceptance checklist covering input validation, normalization order, tokenization, sorting, and the top-10 limit.
- Corrected the reference implementation's comment so it accurately describes how combining marks interact with apostrophe and hyphen joins.

These are follow-up change notes, not part of the original review output.
