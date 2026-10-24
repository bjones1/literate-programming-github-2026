LLM Review Report
=================

<strong>LLM Used:</strong> Google AI for building spec / Claude free version for review
<strong>Prompt:</strong> "Review this spec"

Output From LLM
===============

Overall the spec is tight. I traced all 37 rows of your table by hand against the five steps and found no contradictions. The problems are a few corrupted or ambiguous spots and several decisions that have no example yet.

## Fix these

~~1. **The apostrophe list in step 1 is corrupted.** The Markdown escaping turned two entries into `` \`\` , ´\` ``. I assume you meant the backtick (U+0060) and acute accent (U+00B4), but a reader can't be sure. Spell the set out as code points, and add an example, because another reading gives `can t` and therefore `can: 1`.~~
~~2. **The stop word `'ee'` looks like two Latin e's.** If so, it filters the English token "ee" and never matches `её`, which becomes Cyrillic `ее` after your `ё→е` step. Check with `[hex(ord(c)) for c in 'ee']`. If it should be Cyrillic, add the example `её` → `""`.~~
~~3. **A rendered table can't show Unicode form.** `café` could be U+00E9 or `e` plus U+0301, and the results differ (`caf` vs `cafe` without NFC). Write inputs as escapes (`"caf\u00e9"`) wherever the form matters.~~
4. **Formatting problems:**
   - ~~The 12-word row opens with two backticks and closes with one.~~
   - ~~"empty string" in the table reads like a literal; write `""`.~~
   - ~~The `\&` escapes in headings are stray.~~

## Decisions with no example yet

~~Each row gives a different output under another plausible reading.~~

| Decision | Input | Expected | Other reading it rules out |
|---|---|---|---|
| ~~Apostrophes are normalized before the stop-word check~~ | `It’s THE end` | `end: 1` | ~~Stop-words matched on raw text gives~~ `it's: 1, end: 1` |
| ~~Hyphenated stop-words are matched whole~~ | `to-do the-end list` | `list: 1` | ~~Splitting on hyphens gives~~ `do: 1, end: 1, list: 1` |
| ~~Hyphenated compounds stay one token~~ | `the-cat` | `the-cat: 1` | ~~Splitting gives~~ `cat: 1` |
| ~~Stop-words match whole tokens only~~ | `island theory` | `island: 1, theory: 1` | ~~Prefix filtering gives~~ `""` |
| ~~Doubled apostrophes are not collapsed~~ | `rock''roll` | `rock: 1, roll: 1` | ~~Collapsing gives~~ `rock'roll: 1` |
| ~~NFC runs before filtering~~ | `cafe\u0301` | `caf: 1` | ~~No NFC gives~~ `cafe: 1` |
| ~~NFC for Cyrillic й~~ | `хаи\u0306` | `хай: 1` | ~~No NFC gives~~ `хаи: 1` |
| ~~NFC runs before~~ `ё→е` | `е\u0308лка` | `елка: 1` | ~~Replacing ё first gives~~ `лка: 1` |
| ~~NFC, not NFKC~~ | `\ufb01nal` | `nal: 1` | ~~NFKC gives~~ `final: 1` |
| ~~`errors='ignore'`~~ | `b"hel\xfflo"` | `hello: 1` | ~~`replace` gives~~ `hel: 1, lo: 1` |
| ~~Only exact `str` and `bytes` are accepted~~ | `bytearray(b"hello")` | `""` | ~~Decoding it gives `hello: 1`~~|
| ~~`lower()` quirk~~ | `İstanbul` | `stanbul: 1` |~~ Documents the lossy behavior~~ |

## Judgment calls to confirm

~~- **Hyphens vs apostrophes.** You normalize typographic apostrophes but not typographic hyphens, so `well\u2011known` (non-breaking hyphen) currently splits into `known: 1, well: 1`. Either map U+2010 and U+2011 to `-` or state that they split, and add one row for whichever you choose.~~
~~- **Lossy non-alphabet letters.** `café` → `caf` is intentional, but it also turns `résumé` into `sum`, and chops Latin-script Kazakh (ä, ö, ş…) and Ukrainian ї/є/ґ. Either list this as an explicit non-goal or accept the ghost words.~~
~~- **Small stop-word list.** It has `don't` and `that's` but not `can't` or `she's`, and Russian `был` but not `была`, `есть` or `когда`. Real articles will likely surface filler words in the top 10. Also, Kazakh `бар` will drop the Russian word "bar".~~
~~- **Homoglyphs.** Latin `c` and Cyrillic `с` stay distinct tokens. Your `helloпривет` row covers mixed scripts, but not this case.~~

## Already solid

- **Border rule order.** The border rule gives the same result whether hyphens and apostrophes are checked simultaneously or sequentially, because step 3 never changes letters. You don't need examples for that.
- **Counting and sorting.** `a-b` shows that length counts the hyphen, `Straße` rules out `casefold()`, and `бала әке` rules out code-point sorting.
- **Cutoff.** The 12-word row tests the cutoff at 10.