# Galach query language

Galach is based on a syntax that seems to be the unofficial standard for search query as user input.
You're probably already somewhat familiar with it, as the same basic syntax is used by virtually all
popular web search engines out there. It is also very similar to [Lucene Query Parser syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html),
used by both Solr and Elasticsearch.

## Terms

1. `Word` term is a string not containing whitespace, unless that whitespace is escaped.

    ```
    word
    ```
    ```
    another\ word
    ```

2. `Phrase` term is formed by enclosing words within double quotation marks `"`.

    ```
    "reality exists"
    ```
    ```
    "what's not real doesn't exist"
    ```

3. `User` term is defined by a leading `@` character, followed by at least one alphanumeric or
    underscore character, followed by arbitrary  sequence of alphanumeric characters, hyphens,
    underscores and dots.

    Regular expression:

    ```
    @[a-zA-Z0-9_][a-zA-Z0-9_\-.]*
    ```

    Examples:

    ```
    @joe.watt
    ```
    ```
    @_alice83
    ```
    ```
    @The-Ronald
    ```

4. `Tag` term is defined by a leading `#` character, followed by at least one alphanumeric or
    underscore character, followed by arbitrary sequence of alphanumeric characters, hyphens,
    underscores and dots.

    Regular expression:

    ```
    \#[a-zA-Z0-9_][a-zA-Z0-9_\-.]*
    ```

    Examples:

    ```
    #php
    ```
    ```
    #PHP-7.1
    ```
    ```
    #query_parser
    ```

## Operators

Terms can be combined or modified using binary and unary operators:

1. `Logical and` is a binary operator that combines left and right operands so that both must
    match.

    It comes in two forms: `AND`, `&&`

    In both cases it must be separated from it's operands by whitespace.

    ```
    coffee AND milk
    ```
    ```
    tea && lemon
    ```

2. `Logical or` is a binary operator that combines left and right operands so that at least one of
    them has to match.

    It comes in two forms: `OR`, `||`

    In both cases it must be separated from it's operands by whitespace.

    ```
    potato OR tomato
    ```
    ```
    true || false
    ```

3. `Logical not` is a unary operator that modifies it's operand so that it must not match.

    It comes in two forms: `NOT`, `!`

    When `NOT` form is used, it must be separated from it's operand by whitespace:

    ```
    NOT important
    ```

    When shorthand form `!` is used it must be adjacent to it's operand:

    ```
    !important
    ```

4. `Mandatory` is a unary operator that modifies it's operand so that it must match.
    It's represented by plus sign `+` and must be placed adjacent to it's operand.

    ```
    +coffee
    ```

5. `Prohibited` is a unary operator that modifies it's operand so that it must not match.
    It's represented by minus sign `-` and must be placed adjacent to it's operand.

    ```
    -cake
    ```

### Operator precedence

Unary operators are applied first, followed by binary operators. When it comes to binary operators, `Logical and` precedes `Logical or`:

1. `Logical not`, `Mandatory`, `Prohibited`
2. `Logical and`
3. `Logical or`

## Grouping

Terms and expressions can be grouped using round brackets. A group is processed as a whole.
Following two examples will be processed as the same, since grouping follows operator associativity:

```
one OR NOT two AND three
```
```
one OR ((NOT two) AND three)
```

But you can also use grouping to change the meaning that would follow from operator associativity:

```
(one OR NOT two) AND three
```
```
one OR NOT (two AND three)
```

## Domains

Domain is an abstract category on which the term or group applies. It's defined by prefixing the
term or group with a domain string, followed by a colon `:`. Domain string must start with at least
one alphanumeric or underscore character, and is followed by arbitrary sequence of alphanumeric
characters, hyphens `-` and underscores `_`.

Note that domain cannot be used on `Tag` and `User` terms. These two in fact define implicit domains
of their own.

Regular expression for domain string:

```
[a-zA-Z_][a-zA-Z0-9_\-]*
```

Examples:

```
type:aeroplane
```
```
title:"Language processor"
```
```
description:(wings AND propeller)
```

## Special characters

Characters that are part of the language syntax must be escaped in order not to be recognized as
such by the engine. These are:

- `(` left paren
- `)` right paren
- `+` plus
- `-` minus
- `!` exclamation mark
- `"` double quote
- `#` hash
- `@` at sign
- `:` colon
- `\` backslash
- `␣` blank space

Character used for escaping is backslash `\`:

```
joined\ word
```
```
"escaped \"double quote\""
```
```
escaped \+operator domain\:word \@user \#tag \(and so on\)
```
```
double backslash \\ is a backslash escaped
```

Aside from quotation marks themselves, escaping is not required inside phrases. Since quotes are
used as delimiters, everything between them is taken as-is. Hence, these will be processed as the
same:

```
"+one -two"
```
```
"\+one \-two"
```

In some cases tokenizer will automatically assume that special character is to be interpreted as if
it was escaped. Following pairs will be processed as the same:

1. Colon at the end of a `Word` is considered part of the `Word`

   ```
   word:
   ```
   ```
   word\:
   ```

2. Colon placed after a domain colon is considered part of the `Word`

   ```
   domain:domain:domain
   ```
   ```
   domain:domain\:domain
   ```

3. Domain can't be used on a `Tag` and `User` terms

   ```
   domain:#tag domain:@user
   ```
   ```
   domain:\#tag domain:\@user
   ```

4. Characters used for `Mandatory`, `Prohibited` and shorthand `Logical not` operators can be
   considered part of the `Word`:

   - When placed after domain colon

      ```
      domain:+word domain:-word domain:!word
      ```
      ```
      domain:\+word domain:\-word domain:\!word
      ```

   - When placed in the middle of the word

      ```
      one+two one-two one!two
      ```
      ```
      one\+two one\-two one\!two
      ```

   - When placed at the end of the `Word`

      ```
      one+ two- three!
      ```
      ```
      one\+ two\- three\!
      ```