Top words
=========

We have a "Trending words" box beside each article on a news site. 
And you need to write a function on a Python that will find 10 most common words in anarticle's text, including how many times each one of them appears. 
Articles may be in English, Russian, or Kazakh. 
Consider the case where decomposed unicode or apostrophes may split the words

| Input                      | Expected output         |
| -------------------------- | ----------------------- |
| `cat dog cat bird dog cat` | cat: 3, dog: 2, bird: 1 |
| `along long long stop along` | long: 2, along: 2, stop: 1 |
| `Қазақстан қазақстан мен мир мир привет мир` | мир: 3, қазақстан: 2, мен: 1,привет: 1 | 

Implementation
--------------

* Python 3.
* Name the function `top_words` and put it in `top_words.py`.
