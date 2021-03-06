![JONF icon](jonf-1280x640.png)

# [JONF](#)

Json for cONF

* Docs: [jonf.app](https://jonf.app/)
{:toc}

## [Quick example](#quick-example)

```jonf
# Fictional supercomputer IaC

name - Deep Thought
answer - 42

hardware =
  cores = 42
  eyes =
    left - green
    right - violet

about -
  indented, unquoted,
  and raw - \no special chars

  multiline string here

pets =
  - cat
  - dog
  - turtle  # or tortoise

friends =
  =
    name - Alice
    age = null
  =
    name - Bob
    age = 42

scripts =
  check -
    set -eu  # No more && chains
    DIRS="src tests"
    lint $DIRS
    test $DIRS
```

Equal JSON:

```json
{
  "name": "Deep Thought",
  "answer": "42",
  "hardware": {
    "cores": 42,
    "eyes": {
      "left": "green",
      "right": "violet"
    }
  },
  "about": "indented, unquoted,\nand raw - \\no special chars\n\nmultiline string here",
  "pets": [
    "cat",
    "dog",
    "turtle"
  ],
  "friends": [
    {
      "name": "Alice",
      "age": null
    },
    {
      "name": "Bob",
      "age": 42
    }
  ],
  "scripts": {
    "check": "set -eu\nDIRS=\"src tests\"\nlint $DIRS\ntest $DIRS"
  }
}
```

## [Goals](#goals)

- Make JONF more readable and suitable for configuration files and DSLs than [JSON](https://www.json.org/)
- Keep JONF simple and flawless, unlike [YAML](https://yaml.org/)
- Do not sacrifice JSON features to achieve previous goals, unlike [StrictYAML](https://hitchdev.com/strictyaml/) and [NestedText](https://nestedtext.org/)

## [Motivation](#motivation)

[JSON](https://www.json.org/) is great for API, but not as great for config files and DSLs, that's why alternatives are trying to fill the gap:

1. [ELDF](https://eldf.org/)
1. [ENO](https://eno-lang.org/)
1. [HCL](https://github.com/hashicorp/hcl)
1. [HJSON](https://hjson.github.io/)
1. [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)
1. [HRON](https://github.com/mrange/hron)
1. [ION](https://amzn.github.io/ion-docs/)
1. [JSON5](https://json5.org/)
1. [Jsonnet](https://jsonnet.org/)
1. [NestedText](https://nestedtext.org/)
1. [SDLang](https://sdlang.org/)
1. [StrictYAML](https://hitchdev.com/strictyaml/)
1. [TOML](https://toml.io/en/)
1. [YAML](https://yaml.org/)  
   ![xkcd: Standards](https://imgs.xkcd.com/comics/standards.png)
1. [JONF](https://jonf.app/)

- Disclaimer: I excluded [XML](https://www.w3.org/XML/) from the competition to match [this XKCD image](https://xkcd.com/927/), also XML hardly competes for goal 1 anyway
- Competition is not a bad thing at all - this is how everything evolves, and it shows there is a market for the product
- Most alternatives above, especially the most popular [YAML](https://yaml.org/), are much more [complex](https://yaml.org/spec/1.2.2/) than JSON and/or have other [flaws](https://noyaml.com/) - please see [the great analysis by StrictYAML](https://hitchdev.com/strictyaml/why-not/)
- [The Norway Problem](https://hitchdev.com/strictyaml/why/implicit-typing-removed/) is a trap even so popular YAML [falls](https://docs.gitlab.com/ee/ci/yaml/script.html#use-special-characters-with-script) into
- [StrictYAML](https://hitchdev.com/strictyaml/) and [NestedText](https://nestedtext.org/) solve this problem by assuming all scalars to be strings, and then using optional external schema to convert these strings to the intended type
- This means standard representation of numbers, `true`, `false`, and `null` is no longer supported without external schema
- Going this route we may start using just plain text format because the app should validate it anyway
- While JSON does not have standard representation of dates, times, and so on, to keep it unbloated, existing JSON scalar types are heavily used, so we'd better try to keep them
- JONF solves [The Norway Problem](https://hitchdev.com/strictyaml/why/implicit-typing-removed/) and keeps JSON scalar types in an elegant and terse way

## [Cheatsheet](#cheatsheet)

Key-value separators and array markers:

- `-` is for unquoted strings
- `=` is for all other values

Bash memo:

- `-` is like single quote `'`
- `=` is like double quote `"`

## [Rules](#rules)

- `-` leads to unquoted string on the same line
    - or to indented unquoted multiline string on the next lines
- `=` leads to double-quoted JSON string or another JSON value on the same line
    - or to indented JONF object on the next lines
    - or to indented JONF array on the next lines
- It is possible to introduce values of custom types after `=`, e.g. numerous ISO 8601 dates and times formats, but this is a way either to growth of complexity of standard and each parser, or to interoperability hell with user-provided callbacks to parse custom types, so this feature should not be implemented
- DSL, e.g. [serverless variables](https://www.serverless.com/framework/docs/providers/aws/guide/variables), should be parsed after JONF is parsed, see [example 10. DSL](#10-dsl)
- You can have any indentation style that you want so long as it is <s>black</s> "two spaces", because standardized indentation improves readability and helps to avoid bugs
- For the same reason, exactly one space char ` ` is required:
    - after array marker
    - before key-value separator
    - after key-value separator if the value is on the same line

## [JONF in 10 examples](#jonf-in-10-examples)

### [1. Root value](#1-root-value)

One-line non-indented JSON value is a valid JONF root value, because in a one-line case, JSON is great:

```json
{"some": "object", "key": "value"}

["some", "array", "here"]

"some string"

-3.14

true

false

null
```

JONF object, JONF array, JONF indented unquoted multiline string - are valid root values too, e.g:

```jonf
  indented unquoted
  multiline

  string
```

Equal JSON:

```json
"indented unquoted\nmultiline\n\nstring"
```

More features of unquoted string are explained in the next examples

### [2. JONF array](#2-jonf-array)

```jonf
- Alice in Wonderland
-
  multiline
  string

  here

= "multiline\nstring\n\nhere"
= "  explici\t whitespace \n"
- unquoted is raw - \no special chars"
- great for regex: [\n\r\t]+
- 42
= 42
- -3.14
= -3.14
- true
= true
- false
= false
- null
= null
- []
= []
- {}
= {}
- ""
= ""
```

Equal JSON:

```json
[
  "Alice in Wonderland",
  "multiline\nstring\n\nhere",
  "multiline\nstring\n\nhere",
  "  explici\t whitespace \n",
  "unquoted is raw - \\no special chars\"",
  "great for regex: [\\n\\r\\t]+",
  "42",
  42,
  "-3.14",
  -3.14,
  "true",
  true,
  "false",
  false,
  "null",
  null,
  "[]",
  [],
  "{}",
  {},
  "\"\"",
  ""
]
```

### [3. JONF object](#3-jonf-object)

```jonf
name - Deep Thought
answer - 42
cores = 42
"some - strange = key" - value
42 - keys are always strings
true = "even with =, it affects values only"
```

Equal JSON:

```json
{
  "name": "Deep Thought",
  "answer": "42",
  "cores": 42,
  "some - strange = key": "value",
  "42": "keys are always strings",
  "true": "even with =, it affects values only"
}
```

### [4. Object in object](#4-object-in-object)

```jonf
type - dragon
eyes =
  left - green
  right - violet
```

Equal JSON:

```json
{
  "type": "dragon",
  "eyes": {
    "left": "green",
    "right": "violet"
  }
}
```

### [5. Object in array](#5-object-in-array)

```jonf
=
  name - Alice
  age = null
=
  name - Bob
  age = 42
```

Equal JSON:

```json
[
  {
    "name": "Alice",
    "age": null
  },
  {
    "name": "Bob",
    "age": 42
  }
]
```

Please note `=` should be used as an array marker, unless you want unquoted strings:

```jonf
- unquoted string
-
  multiline - unquoted
  string = here
```

Equal JSON:

```json
[
  "unquoted string",
  "multiline - unquoted\nstring = here"
]
```

### [6. Depth and boundaries](#6-depth-and-boundaries)

JONF is arguably more readable than YAML below, let alone YAML's typing flaw:

```yaml
person:
  name: abcd
  nick: efgh
friends:
- name: hijk
  nick: lmno
- name: pqrs
  nick: tuvw
```

It looks like all names here are on the same depth in YAML, which is misleading, and it takes some effort to notice the boundaries of each friend

JONF shows the depth and boundaries correctly, improving readability:

```jonf
person =
  name - abcd
  nick - efgh
friends =
  =
    name - hijk
    nick - lmno
  =
    name - pqrs
    nick - tuvw
```

### [7. Array in object](#7-array-in-object)

```jonf
name - Bob
kids =
  - Charlie
  - Dave
  - Eve
```

Equal JSON:

```json
{
  "name": "Bob",
  "kids": [
    "Charlie",
    "Dave",
    "Eve"
  ]
}
```

### [8. Array in array](#8-array-in-array)

```jonf
=
  - We
  - are
=
  - almost
  =
    - done!
```

Equal JSON:

```json
[
  [
    "We",
    "are"
  ], [
    "almost",
    [
      "done!"
    ]
  ]
]
```

JONF is obviously better than JSON in a multi-line case

Again, JSON is the best for one-liners, that's why they are valid in JONF:

```json
[["We", "are"], ["almost", ["done!"]]]
```

### [9. Comment](#9-comment)

Text from `#` to the end of line is a comment if `#` is the first character in the line or there is a whitespace before it and it is not inside of JSON value:

```jonf
# Full-line comment
name - Alice  # Inline comment
url - https://example.org/#alice
location = "Wonderland # 42"
```

Equal JSON:

```json
{
  "name": "Alice",
  "url": "https://example.org/#alice",
  "location": "Wonderland # 42",
}
```

JONF uses `#` and never `//` or `/* */` for the same reason JSON uses double-quotes and never single-quotes: to keep it simple

### [10. DSL](#10-dsl)

In `serverless.jonf` [variables](https://www.serverless.com/framework/docs/providers/aws/guide/variables) are parsed after JONF is parsed:

```jonf
custom =
  debug = true
  verbose - ${self:custom.debug}
```

Equal JSON:

```json
{
  "custom": {
    "debug": true,
    "verbose": "${self:custom.debug}"
  }
}
```

Equal JSON, parsed by DSL:

```json
{
  "custom": {
    "debug": true,
    "verbose": true
  }
}
```

Note that DSL decided to change type of `verbose` to the type of the variable it referenced. JONF never does that, it is just a container format, strictly mapped to JSON only

## [Roadmap](#roadmap)

- Add JONF to [github/linguist](https://github.com/github/linguist/blob/master/lib/linguist/languages.yml) and [Rouge](https://github.com/rouge-ruby/rouge) highlighters to simplify the reviews
- Reviews and fixes of this draft specification
- Formal grammar importing [JSON grammar](http://crockford.com/mckeeman.html) but excluding newlines from its `ws` rule
- Apply this grammar to the reference [JONF parser/formatter in Python](https://pypi.org/project/jonf/)
- Bump to version 1.0.0
- Get contributed JONF parsers/formatters for other languages

## [JONF as JONF](#jonf-as-jonf)

```jonf
Name - JONF
Version - 0.0.17
Filename extension - .jonf
Internet media type - application/jonf  # TODO
Website - jonf.app
Email - support@jonf.app
Maintainer - whyolet.com
License - MIT
Open format? - Yes
Type of format - Data interchange
Extended from - JSON
```

## [That's all!](#thats-all)

Simple and valuable, right?

Thank you for reading

Please [post an issue or idea](https://github.com/whyolet/jonf/issues) or contribute otherwise:

<img src="GitHub-Mark-32px.png" alt="GitHub icon" align="center" /> [https://github.com/whyolet/jonf](https://github.com/whyolet/jonf)
