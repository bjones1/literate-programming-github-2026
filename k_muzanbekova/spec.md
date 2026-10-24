Top words
=========

A news site shows a "Trending words" box beside each article. Write a Python
function that finds the 10 most common words in an article's text, along with
how many times each one appears. Articles may be in English, Russian, or Kazakh.

Implementation
--------------

* Python 3.
* Name the function `top_words` and put it in `top_words.py`.

Assignment
----------

> "The model's job is to produce something plausible. Your job is to make sure
> only one thing is plausible." -- Claude

Tighten this specification to produce one plausible result. Then, for each
decision your spec makes, describe at least one input and its expected output,
usually as a table in your spec. A useful example gets a different output under
some other plausible reading of the original spec. For example:

| Input                         | Expected output            |
| ----------------------------- | -------------------------- |
| `cat dog cat bird dog cat`    | cat: 3, dog: 2, bird: 1    |
| `dog cat bird`                | bird: 1, cat: 1, dog: 1    |
| `Word word WORD`              | word: 3                    |
| `the cat and a dog`           | cat: 1, dog: 1             |
| `can't full-time`             | can't: 1, full-time: 1     |
| `"co-worker... or --friend?"` | co-worker: 1, friend: 1    |
| `apple 456-apple`             | apple: 2                   |
| `rock 'n' roll`               | rock: 1, roll: 1           |
| `ПРИВЕТ HELLO`                | hello: 1, привет: 1        |
| `A I b d c e hello`           | hello: 1                   |
| `алма және анар мен банан`    | алма: 1, анар: 1, банан: 1 |
| `can't can’t`                 | can't: 2                   |
| `abc123def`                   | abc: 1, def: 1             |
| `a-b`                         | a-b: 1                     |
| `foo-1bar`                    | bar: 1, foo: 1             |
| `осы прилетели`               | прилетели: 1               |
| `бала әке`                    | әке: 1, бала: 1            |
| `well--known`                 | known: 1, well: 1          |
| `well-known`                  | well-known: 1              |
| `wellness well-known`         | well-known: 1, wellness: 1 |
| `can't cant`                  | can't: 1, cant: 1          |
| `cat cats`                    | cat: 1, cats: 1            |
| `елка ёлка`                   | елка: 2                    |
| ``ab cd ef gh ij kl mn op qr st uv wx` | ab: 1, cd: 1, ef: 1, gh: 1, ij: 1, kl: 1, mn: 1, op: 1, qr: 1, st: 1|
| `café`                        | caf: 1                     |
| `None`, `""`, `"    "`        | empty string               |
| `True`                        | empty string               |
| `['cat', 'dog']`              | empty string               |
| `---`                         | empty string               |
| `что не он всё`               | empty string               |
| `helloпривет`                 | helloпривет: 1             |
| `It's THE end`                | end: 1                     |
| `‘hi’ ʼhiʼ`                   | hi: 2                      |
| `rock'n'roll dogs'`           | dogs: 1, rock'n'roll: 1    |
| `b"hello world"`              | hello: 1, world: 1         |
| `Straße`                      | stra: 1                    |
| `cat😀dog`                   | cat: 1, dog: 1             |

Specification: Top Words
========================

Program Description
-------------------

This program processes an input text document, filters out common English, Russian, and Kazakh stop-words, and identifies the 10 most frequently occurring words. The platform officially supports only standard English, Russian, and Kazakh languages. 

`VALID_ALPHABET = "'-abcdefghijklmnopqrstuvwxyzаәбвгғдежзийкқлмнңоөпрстуұүфхһцчшщъыіьэюя"`

Inputs \& Interface
-------------------

* **Function:** `top_words(text: str) -> str`
* **text (string):** Text content to analyze.

Outputs
-------

* **Format**: Returns a single lowercase string formatted as `word: count, word: count`. Pairs are separated by a comma and a space. 
The output string contains **no leading or trailing whitespaces**.
* **Sorting Rules:** Sorted primarily by descending frequency. If frequencies are equal, tokens are sorted lexicographically based strictly on the character indices in `VALID_ALPHABET` (where `'` is 0, `-` is 1, `a` is 2, etc.). Shorter prefix words sort before longer words.  
* The function returns a maximum of 10 words. If the number of unique valid words in the text is less than 10, it returns all found words.

Constraints \& Order of Operations
---------------------------

* **Normalization:**
	* If `text` is `bytes`, decode to a string using `text.decode('utf-8', errors='ignore')`.
  * If `text` is not a `str` or `bytes`, treat it as an empty string `""`.

* **Order of Operations:**
  1. **Clean Text:** Convert to lowercase using `str.lower()`, normalize Unicode (NFC), and replace all typographic/smart/modifier apostrophe variants (`’`, `‘`, `ʼ`, `ʽ`, `ʹ`, `′`, \`\` , ´\`, `‛`, `＇`) with a standard straight apostrophe (`'`). Replace the Russian letter `ё`  with  `е`.
  2. **Remove garbage:** Replace all digits, special characters, emojis, punctuation marks and **any alphabetic characters/letters not explicitly included in `VALID_ALPHABET`** with spaces.
  3. **Fix Borders:** Inspect every hyphen (`-`) and apostrophe (`'`). It is kept **only if both its immediate left and right neighbors are alphabet letters** (indices 2 or higher in `VALID_ALPHABET`). Otherwise, replace it with a space.
  4. **Tokenize & Filter:** Split by whitespace. Discard any token that is shorter than 2 characters or is present in `DEFAULT_STOP_WORDS`.
  5. **Count & Top 10**: Compute raw frequencies, sort according to Sorting Rules, and slice the top 10 items.

Stop-words list (English, Russian, Kazakh) that the function should ignore during counting.
----------------------------

`DEFAULT_STOP_WORDS` = {
'the', 'an', 'and', 'or', 'but', 'in', 'on','at',
'to', 'for', 'of', 'with', 'is', 'are', 'was', 'were', 
'it', 'this', 'that', 'you', 'he', 'she', 'they', 'i'm', "it's", "he's", "they're", 'to-do', 'from', 'has', 'have', 'been', 'by', 'as', 'his', 'her', 'their', 'will', 'not', 'the-end', "don't", "that's", "we're",
'be', 'we', 'my', 'if',  

'не', 'что', 'он', 'она', 'они', 'на', 'мы', 'ли', 'бы', 'то', 'но', 'из', 
'от', 'по', 'как', 'так', 'его', 'для', 'или', 'ты', 'вас', 'мне', 'это', 'за', 'же', 'да', 'все', 'был', 'уже', 'ee', 'вы',

'мен', 'пен', 'бен', 'және', 'үшін', 'осы', 'бұл', 'сол', 'деп', 
'ғой', 'ма', 'ме', 'ба', 'бе', 'па', 'пе', 'әр', 'бір', 'жоқ', 'бар', 'ол', 'де', 'емес', 'бірақ'
}