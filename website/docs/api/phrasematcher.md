---
title: PhraseMatcher
teaser: Match sequences of tokens, based on documents
tag: class
source: spacy/matcher/phrasematcher.pyx
new: 2
---

The `PhraseMatcher` lets you efficiently match large terminology lists. While
the [`Matcher`](/api/matcher) lets you match sequences based on lists of token
descriptions, the `PhraseMatcher` accepts match patterns in the form of `Doc`
objects. See the [usage guide](/usage/rule-based-matching#phrasematcher) for
examples.

## PhraseMatcher.\_\_init\_\_ {#init tag="method"}

Create the rule-based `PhraseMatcher`. Setting a different `attr` to match on
will change the token attributes that will be compared to determine a match. By
default, the incoming `Doc` is checked for sequences of tokens with the same
`ORTH` value, i.e. the verbatim token text. Matching on the attribute `LOWER`
will result in case-insensitive matching, since only the lowercase token texts
are compared. In theory, it's also possible to match on sequences of the same
part-of-speech tags or dependency labels.

If `validate=True` is set, additional validation is performed when pattern are
added. At the moment, it will check whether a `Doc` has attributes assigned that
aren't necessary to produce the matches (for example, part-of-speech tags if the
`PhraseMatcher` matches on the token text). Since this can often lead to
significantly worse performance when creating the pattern, a `UserWarning` will
be shown.

> #### Example
>
> ```python
> from spacy.matcher import PhraseMatcher
> matcher = PhraseMatcher(nlp.vocab)
> ```

| Name                                    | Description                                                                                            |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `vocab`                                 | The vocabulary object, which must be shared with the documents the matcher will operate on. ~~Vocab~~  |
| `attr` <Tag variant="new">2.1</Tag>     | The token attribute to match on. Defaults to `ORTH`, i.e. the verbatim token text. ~~Union[int, str]~~ |
| `validate` <Tag variant="new">2.1</Tag> | Validate patterns added to the matcher. ~~bool~~                                                       |

## PhraseMatcher.\_\_call\_\_ {#call tag="method"}

Find all token sequences matching the supplied patterns on the `Doc` or `Span`.

> #### Example
>
> ```python
> from spacy.matcher import PhraseMatcher
>
> matcher = PhraseMatcher(nlp.vocab)
> matcher.add("OBAMA", [nlp("Barack Obama")])
> doc = nlp("Barack Obama lifts America one last time in emotional farewell")
> matches = matcher(doc)
> ```

| Name                                  | Description                                                                                                                                                                                                                                                                                              |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `doclike`                             | The `Doc` or `Span` to match over. ~~Union[Doc, Span]~~                                                                                                                                                                                                                                                  |
| _keyword-only_                        |                                                                                                                                                                                                                                                                                                          |
| `as_spans` <Tag variant="new">3</Tag> | Instead of tuples, return a list of [`Span`](/api/span) objects of the matches, with the `match_id` assigned as the span label. Defaults to `False`. ~~bool~~                                                                                                                                            |
| **RETURNS**                           | A list of `(match_id, start, end)` tuples, describing the matches. A match tuple describes a span `doc[start:end`]. The `match_id` is the ID of the added match pattern. If `as_spans` is set to `True`, a list of `Span` objects is returned instead. ~~Union[List[Tuple[int, int, int]], List[Span]]~~ |

<Infobox title="Note on retrieving the string representation of the match_id" variant="warning">

Because spaCy stores all strings as integers, the `match_id` you get back will
be an integer, too – but you can always get the string representation by looking
it up in the vocabulary's `StringStore`, i.e. `nlp.vocab.strings`:

```python
match_id_string = nlp.vocab.strings[match_id]
```

</Infobox>

## PhraseMatcher.\_\_len\_\_ {#len tag="method"}

Get the number of rules added to the matcher. Note that this only returns the
number of rules (identical with the number of IDs), not the number of individual
patterns.

> #### Example
>
> ```python
>   matcher = PhraseMatcher(nlp.vocab)
>   assert len(matcher) == 0
>   matcher.add("OBAMA", [nlp("Barack Obama")])
>   assert len(matcher) == 1
> ```

| Name        | Description                  |
| ----------- | ---------------------------- |
| **RETURNS** | The number of rules. ~~int~~ |

## PhraseMatcher.\_\_contains\_\_ {#contains tag="method"}

Check whether the matcher contains rules for a match ID.

> #### Example
>
> ```python
>   matcher = PhraseMatcher(nlp.vocab)
>   assert "OBAMA" not in matcher
>   matcher.add("OBAMA", [nlp("Barack Obama")])
>   assert "OBAMA" in matcher
> ```

| Name        | Description                                                    |
| ----------- | -------------------------------------------------------------- |
| `key`       | The match ID. ~~str~~                                          |
| **RETURNS** | Whether the matcher contains rules for this match ID. ~~bool~~ |

## PhraseMatcher.add {#add tag="method"}

Add a rule to the matcher, consisting of an ID key, one or more patterns, and a
callback function to act on the matches. The callback function will receive the
arguments `matcher`, `doc`, `i` and `matches`. If a pattern already exists for
the given ID, the patterns will be extended. An `on_match` callback will be
overwritten.

> #### Example
>
> ```python
>   def on_match(matcher, doc, id, matches):
>       print('Matched!', matches)
>
>   matcher = PhraseMatcher(nlp.vocab)
>   matcher.add("OBAMA", [nlp("Barack Obama")], on_match=on_match)
>   matcher.add("HEALTH", [nlp("health care reform"), nlp("healthcare reform")], on_match=on_match)
>   doc = nlp("Barack Obama urges Congress to find courage to defend his healthcare reforms")
>   matches = matcher(doc)
> ```

<Infobox title="Changed in v3.0" variant="warning">

As of spaCy v3.0, `PhraseMatcher.add` takes a list of patterns as the second
argument (instead of a variable number of arguments). The `on_match` callback
becomes an optional keyword argument.

```diff
patterns = [nlp("health care reform"), nlp("healthcare reform")]
- matcher.add("HEALTH", on_match, *patterns)
+ matcher.add("HEALTH", patterns, on_match=on_match)
```

</Infobox>

| Name           | Description                                                                                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `match_id`     | An ID for the thing you're matching. ~~str~~                                                                                                               |  |
| `docs`         | `Doc` objects of the phrases to match. ~~List[Doc]~~                                                                                                       |
| _keyword-only_ |                                                                                                                                                            |
| `on_match`     | Callback function to act on matches. Takes the arguments `matcher`, `doc`, `i` and `matches`. ~~Optional[Callable[[Matcher, Doc, int, List[tuple], Any]]~~ |

## PhraseMatcher.remove {#remove tag="method" new="2.2"}

Remove a rule from the matcher by match ID. A `KeyError` is raised if the key
does not exist.

> #### Example
>
> ```python
> matcher = PhraseMatcher(nlp.vocab)
> matcher.add("OBAMA", [nlp("Barack Obama")])
> assert "OBAMA" in matcher
> matcher.remove("OBAMA")
> assert "OBAMA" not in matcher
> ```

| Name  | Description                       |
| ----- | --------------------------------- |
| `key` | The ID of the match rule. ~~str~~ |
