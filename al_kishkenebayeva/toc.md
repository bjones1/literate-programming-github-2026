Top words
=========

A news site shows a "Trending words" box beside each article. Write a Python
function that finds the 10 most common words in an article's text, along with
how many times each one appears. Articles may be in English, Russian, or Kazakh.

Implementation
--------------

* Python 3, standard library only.
* Name the function `top_words` and put it in `top_words.py`.
* Signature: `top_words(text: str) -> str`

Rules
-----

* <strong>Letter:</strong> any character for which `str.isalpha()` is true (any
  script, including Kazakh ә ғ қ ң ө ұ ү һ і and accented Latin).
* <strong>Word:</strong> a run of letters. A hyphen `-` or apostrophe (`'` or
  `’`) belongs to the word only when it sits directly between two letters
  (`well-known`, `don't`, `rock'n'roll`). Anywhere else it is a separator
  (`hello--world` → `hello`, `world`; `-hello`, `hello-` → `hello`).
* <strong>Separators:</strong> everything else: digits, underscores,
  punctuation, whitespace.
* <strong>Case:</strong> words are lowercased with `str.lower()`. No other
  normalization (`don't` and `don’t` are different words).
* <strong>No filtering:</strong> stop words (`the`, `и`, `және`) are counted
  like any other word.
* <strong>Ranking:</strong> count descending; equal counts ordered by first
  appearance. Apply this to all distinct words, then keep the first 10.
* <strong>Output:</strong> `word: count` pairs joined by `,` (a comma followed
  by one space), with no leading or trailing whitespace. Fewer than 10 distinct
  words → return all. No words → empty string.

Assignment
----------

Every input in the tables below is a Python expression (a string literal or an
expression that builds a string, e.g. `"a " * 3`). Evaluate it to get the `text`
argument, then call `top_words(text)`. Expected outputs are the exact returned
string, without the surrounding backticks.

### Basic

| Input                        | Expected output           |
| ---------------------------- | ------------------------- |
| `"cat dog cat bird dog cat"` | `cat: 3, dog: 2, bird: 1` |
| `"Кот, кот! Пёс и кот."`     | `кот: 3, пёс: 1, и: 1`    |
| `"Қала қала, ауыл."`         | `қала: 2, ауыл: 1`        |
| `"Don't stop, don't"`        | `don't: 2, stop: 1`       |

### Top-10 cutoff

| Input                                                                                                                              | Expected output                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `"a " * 12 + "b " * 11 + "c " * 10 + "d " * 9 + "e " * 8 + "f " * 7 + "g " * 6 + "h " * 5 + "i " * 4 + "j " * 3 + "k " * 2 + "l "` | `a: 12, b: 11, c: 10, d: 9, e: 8, f: 7, g: 6, h: 5, i: 4, j: 3` |
| `"a b c d e f g h i j"`                                                                                                            | `a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1, j: 1`    |
| `"a b c d e f g h i j k"`                                                                                                          | `a: 1, b: 1, c: 1, d: 1, e: 1, f: 1, g: 1, h: 1, i: 1, j: 1`    |

### Ties

Equal counts are ordered by first appearance, not alphabetically.

| Input                                                                                                    | Expected output                                                             |
| -------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `"b a a b"`                                                                                              | `b: 2, a: 2`                                                                |
| `"b a b a c c"`                                                                                          | `b: 2, a: 2, c: 2`                                                          |
| `"a " * 10 + "b " * 9 + "c " * 8 + "d " * 7 + "e " * 6 + "f " * 5 + "g " * 4 + "h " * 3 + "z x y z x y"` | `a: 10, b: 9, c: 8, d: 7, e: 6, f: 5, g: 4, h: 3, z: 2, x: 2`               |
| `"кот мысық кот cat Cat мысық кот собака dog cat a b c d e f"`                                           | `кот: 3, cat: 3, мысық: 2, собака: 1, dog: 1, a: 1, b: 1, c: 1, d: 1, e: 1` |

* Third row: `z`, `x`, `y` tie for the last two slots. `y` appeared last, so it
  is dropped.
* Fourth row: 11 distinct words. `кот` and `cat` tie at 3, and `f` is dropped at
  the cutoff.

### Realistic article

| Input                                                                                                                                                                                    | Expected output                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `"The city council approved the new budget. The budget includes funds for the park, the school, and the library. Council members said the budget is the largest in the city's history."` | `the: 9, budget: 3, council: 2, city: 1, approved: 1, new: 1, includes: 1, funds: 1, for: 1, park: 1` |

`city's` is a separate word, so `city` is counted once.

### Case and languages

| Input                                      | Expected output                                        |
| ------------------------------------------ | ------------------------------------------------------ |
| `"Привет привет ПРИВЕТ мир Мир"`           | `привет: 3, мир: 2`                                    |
| `"Қала ҚАЛА қала"`                         | `қала: 3`                                              |
| `"A a Á á"`                                | `a: 2, á: 2`                                           |
| `"ә ғ қ ң ө ұ ү һ і"`                      | `ә: 1, ғ: 1, қ: 1, ң: 1, ө: 1, ұ: 1, ү: 1, һ: 1, і: 1` |
| `"the cat и кот және мысық the cat и кот"` | `the: 2, cat: 2, и: 2, кот: 2, және: 1, мысық: 1`      |

### Word boundaries

| Input                                    | Expected output                      |
| ---------------------------------------- | ------------------------------------ |
| `"well-known well-known 'quoted' don't"` | `well-known: 2, quoted: 1, don't: 1` |
| `"hello-world hello world"`              | `hello-world: 1, hello: 1, world: 1` |
| `"hello--world -hello hello-"`           | `hello: 3, world: 1`                 |
| `"rock'n'roll rock'n'roll"`              | `rock'n'roll: 2`                     |
| `"foo_bar foo bar"`                      | `foo: 2, bar: 2`                     |
| `"abc123 123abc"`                        | `abc: 2`                             |
| `"abc123 abc 123"`                       | `abc: 2`                             |

### Empty and large input

| Input                 | Expected output         |
| --------------------- | ----------------------- |
| `""`                  | <em>(empty string)</em> |
| `"   "`               | <em>(empty string)</em> |
| `"42 !!! ..."`        | <em>(empty string)</em> |
| `"word " * 1_000_000` | `word: 1000000`         |

### Performance

The `"word " * 1_000_000` case must complete in **under 2 seconds**, measured as
the wall-clock time of the single `top_words(text)` call (excluding construction
of the input string), on CPython 3.10 or newer running on a typical modern
laptop or desktop CPU (single thread). A straightforward linear-time
implementation typically finishes in well under one second.
