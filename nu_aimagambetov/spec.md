Top words
=========

A news site shows a "Trending words" box beside each article. Write a Python
function that finds the 10 most common words in an article's text, along with
how many times each one appears. Articles may be in English, Russian, or Kazakh.

Implementation
--------------

* Use python3 and its standard library.

* Function signature is `top_words(text: str) -> OrderedDict`,
  implement it in `top_words.py`.

* A word can only contain letters from the following character sets:

Latin: `"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"`,

Cyrillic:
`"абвгдежзийклмнопрстуфхцчшщъыьэюяёАБВГДЕЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯЁәғқңөұүһіӘҒҚҢӨҰҮҺІ"`

* Perform unicode NFC normalization. All characters that are not in the list of
  the characters above are treated as word delimiters.

* Only Cyrillic-script Kazakh is supported, and Kazakh-specific Latin characters
  are treated as delimiters.

* Split the text into tokens using the delimiters. A token is a maximal nonempty
  run of allowed characters.

* After splitting the text into tokens, convert each token into lowercase using
  str.lower(). A token is considered a valid word if it only contains Latin
  characters (defined above), or Cyrillic characters (defined above), but not
  both. Tokens that are not valid are discarded entirely.

* Use a Python dict to store valid word frequencies, where each key is a word
  and its value is the number of occurrences.

* Sort words by tuple (-frequency, word) in ascending order, select at most 10
  words, return the results as collections.OrderedDict, where word is the key,
  and the value is the frequency. (-frequency, word) is used to first sort words
  by frequency, and break ties by comparing words by unicode code point.

* Raise TypeError if the text is not str

* Because the word is defined as a token, various special cases of words are not
  treated differently, including: hyphenated compound words, words containing
  apostrophes, etc.

Examples
--------

| Input                                  | Expected output                                                                               |
| -------------------------------------- | --------------------------------------------------------------------------------------------- |
| `"cat dog cat bird dog cat"`           | OrderedDict({"cat": 3, "dog": 2, "bird": 1})                                                  |
| `"hello hEllo helLO HellO"`            | OrderedDict({"hello": 4})                                                                     |
| `"-учебно-методический-"`              | OrderedDict({"методический": 1, "учебно": 1})                                                 |
| `":<>\_<>$934"`                        | OrderedDict({})                                                                               |
| `"учебно--методический"`               | OrderedDict({"методический": 1, "учебно": 1})                                                 |
| `"hello>world+hello"`                  | OrderedDict({"hello": 2, "world": 1})                                                         |
| `"students'"`                          | OrderedDict({"students": 1})                                                                  |
| `"hello666world"`                      | OrderedDict({"hello": 1, "world": 1})                                                         |
| "`helloпривет hello привет"`           | OrderedDict({"hello": 1, "привет": 1})                                                        |
| `"CoViD-19"`                           | OrderedDict({"covid": 1})                                                                     |
| "`сәлемпривет"`                        | OrderedDict({"сәлемпривет": 1})                                                               |
| `"ё е\u0308еее"`                       | OrderedDict({"ё": 1, "ёеее": 1})                                                              |
| `""`                                   | OrderedDict({})                                                                               |
| `"a1 b2 c3 d4 e5 f6 g7 h8 i9 j10 k11"` | OrderedDict({"a": 1, "b": 1, "c": 1, "d": 1, "e": 1, "f": 1, "g": 1, "h": 1, "i": 1, "j": 1}) |
| `"İstanbul"`                           | OrderedDict({"stanbul": 1})                                                                   |
| `"don't doesn't"`                      | OrderedDict({"t": 2, "don": 1, "doesn": 1})                                                   |
