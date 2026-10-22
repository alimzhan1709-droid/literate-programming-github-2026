Prompting GPT-6-luna, optimized for balance. Conclusion after the review: there are minor issues to be fixed that can greatly improve specification (e.g. input should be all in quotation, take into account the space after comma, runtime expectation should be measurable).



# Specification review: `top-words.md`

The specification is detailed and its examples cover ranking, ties, Unicode
letters, punctuation boundaries, and empty input well. A few details should be
made more precise before implementation:

1. **Input notation is inconsistent.** The assignment says inputs are Python
   expressions evaluated before the call, but many examples are bare text in
   code formatting (for example, `cat dog cat bird dog cat`), which is not a
   valid Python expression. Clarify that unquoted table entries are literal
   text while quoted expressions are evaluated, or quote every literal input.
   This matters for anyone implementing an automated test harness.
2. **Specify the exact output delimiter.** The rule says pairs are joined by
   `,`, while the expected outputs use comma followed by a space (`, `).
   State the latter explicitly, for example: “Join pairs with `, `.”
3. **Make the runtime expectation measurable or advisory.** “About a second”
   for one million words depends on the machine and test environment. Either
   define the benchmark environment and a pass threshold, or describe this as
   an optimization expectation rather than a strict acceptance criterion.

These are clarification requests, not correctness problems with the listed
expected outputs.