# Top words specification
Function top_words(text: str) -> list[tuple[str, int]] returns the 10 most frequent words in an article's text, along with their occurrence counts.

Case is ignored: Cat, CAT, cat are the same word.

Diacritics and native alphabet characters (Latin, Cyrillic, Kazakh ә, ғ, қ, ң, ө, ұ, ү, һ, і, etc.) are preserved as-is; no transliteration, stripping of accents, or stemming/lemmatization is performed.

Word forms are not merged: cat and cats are different words.

# Counting and Sorting

    Count the occurrences of each normalized word.

    Sort the unique words primarily by descending frequency.

    On frequency ties, sort secondarily ascending by Unicode code point (standard lexicographic string comparison).

    Return the top 10 (word, count) pairs.

    If there are fewer than 10 unique words, return all of them.

    If the input text is empty or contains no alphabetic characters, return an empty list [].

# Output Format

A list of (str, int) tuples (e.g., [("word", 3), ("test", 1)]).

Not a plain string, not a dictionary.

Input                          | Expected output               |       What it pins down
-------------------------------|-------------------------------|--------------------------------------------------------------
"" and "123 !!! ---"	       |   []	                       |      Empty output for empty string or text without letters
12	"x " * 15 + "y"	           |   [("x", 15), ("y", 1)]	   |      Output truncation to 10 items when >10 unique words exist
"Привет привет ПРИВЕТ мир"	   |   ("привет",3),("мир",1)      |      Cyrillic + case
"Сәлем сәлем Қазақстан"	       |   ("сәлем",2),("қазақстан",1) |	  Kazakh letters, lowercasing
"hello, hello! hello?"	       |   ("hello",3)	               |      Punctuation is a separator