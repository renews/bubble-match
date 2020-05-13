# BubbleExpr

NLP rule language for matching natural language against a rule base.

**Warning: this project is currently heavily in development. Its syntax and API might change at any time**

[Check out the demo environment](http://expr-ninja.apps.botsqd.com/) powered by Phoenix Liveview.


## Examples

Matching basic sequences of words

| Match string  | Example           | Matches? |
|---------------|-------------------|----------|
| `hello world` | Hello, world!     | **yes**  |
| `hello world` | Well hello world  | **yes**  |
| `hello world` | hello there world | no       |
| `hello world` | world hello       | no       |


Matching regular expressions

| Match string | Example | Matches? |
|--------------|---------|----------|
| `/[a-z]+/`   | abcd    | **yes**  |


Match entities, with the help of Spacy and Duckling preprocessing and
tokenizing the input:

| Match string | Matches                         | Does not match  |
|--------------|---------------------------------|-----------------|
| `[PERSON]`   | George Baker                    | Hello world     |
| `[time]`     | I walked to the store yesterday | My name is John |


## Rules overview

The match syntax is composed of adjacent and optionally nested,
rules. Each individual has the following syntax:

- Basic words; only alphanumeric characters and the quote characters
  - matching is done on both the lowercased, normalized version of the
    word, and on the lemmatization of the word.

  - use a dash (`-`) to match on compound nouns: `was-machine` matches
    all of `wasmachine`, `was-machine` and `was machine`.

- `"Literal word sequence"`
  - Matches a literal piece of text, possibly spread out over multiple tokens.

- `_` without any range specifier, matches 0-5 of any available token, greedy.

- Stand-alone range specifier
  - `[1]` match exactly one token; any token
  - `[2+]` match 2 or more tokens (greedy)
  - `[1-3]` match 1 to 3 tokens (greedy)
  - `[2+?]` match 2 or more tokens (non-greedy)
  - `[1-3?]` match 1 to 3 tokens (non-greedy)

- Entity tokens: `[email]` matches a token of type `:entity` with
  value.kind == `email`. Entities are extracted by external means,
  e.g. by an NLP NER engine like Duckling.

  Entities are automatically captured under a variable with the same
  name as the entity's kind.

- Regex tokens: `[/regex/]` matches the given regex against the raw text in the token

- OR / grouping construct

  - `pizza | fries | chicken` - OR-clause on the root level without
    parens, matches either token

  - `a ( a | b | c )` - use parentheses to separate OR-clauses;
    matches one token consisting of first `a`, and then `a`, `b` or
    `c`.

  - `( a )[3+]` matches 3 or more token consisting of `a`
  - `( hi | hello )[=greeting]` matches 1 token and stores it in `greeting`

- Permutation construct
 - `< a b c >` matches any permutation of the sequence `a b c`; `a c b`, or `b a c`, or `c a b`, etc

- Start / end sentence markers
- `[Start]` Matches the start of a sentence
- `[End]` Matches the end of a sentence

- Word collections ("concepts")
  - `@food` matches any token in the `food` collection.
  - `@food.subcat` matches any token in the given subcategory.

  Concept compilation is done as part of the parse phase; the concepts
  compiler must must return an `{m, f, a}` triple. In runtime, this
  MFA is called while matching, and thus, it must be a fast function.

- Part-of-speech tags (word kinds), e.g.
  - `%VERB` matches any verb
  - `%NOUN` matches any noun
  - Any other POS Spacy tags are valid as well


### Rule modifiers

Any rule can have a `[]` block which contains a repetition modifier
and/or a capture expression.

Entity blocks are automatically captured as the entity kind.

### Sentences

The expression matching works on a per-sentence basis; the idea is
that it does not make sense to create expressions that span over
sentences.

The builtin sentence tokenizer (`BubbleExpr.Sentence.Tokenizer`) does
**not** have the concept of sentences, and thus treats each input as a
single sentence, even in the existence of periods in the input.

However, the prefered way of using this library is by running the
input through an NLP preprocessor like Spacy, which does tokenize an
input into individual sentences.





## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `bubble_expr` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:bubble_expr, "~> 0.1.0"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/bubble_expr](https://hexdocs.pm/bubble_expr).
