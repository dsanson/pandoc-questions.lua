# pandoc-questions.lua

A pandoc lua filter for managing different versions in a single
document. Specifically, I designed it for managing multiple versions of the
same quiz or exam.

For example, suppose you want to write three versions of a quiz, each with the
same first two questions, but a different third question:

``` markdown
@.  What is your name?
@.  What is your quest?
@.  [q1] What is your favorite color?
@.  [q2] What is the capital of Assyria?
@.  [q3] What is the air-speed velocity of an unladen swallow? 
```

The last three questions begin with a "version tag"---a single word inside square
brackets. To produce a specific version of the quiz, specify that version as
metadata:

``` sh
> pandoc quiz.md -t markdown --lua-filter pandoc-questions.lua --metadata version:q3
1.  What is your name?
2.  What is your quest?
3.  What is the air-speed velocity of an unladen swallow? 
```

As you can see, when a version is specified, items with non-matching version
tags are filtered out. Also, the version tag is removed.

If no version is specified, or if the version is set to "nil" using
`--metadata version:nil`, everything is passed 
through unfiltered, including the version tags:

```sh
> pandoc quiz.md -t markdown --lua-filter pandoc-questions.lua
1.  What is your name?
2.  What is your quest?
3.  \[q1\] What is your favorite color?
4.  \[q2\] What is the capital of Assyria?
5.  \[q3\] What is the air-speed velocity of an unladen swallow?
```

If the version is set to "all" using `--metadata version:all`, everything is passed through unfiltered, and all
the version tags are deleted:

```sh
> pandoc quiz.md -t markdown --lua-filter pandoc-questions.lua --metadata version:all
1.  What is your name?
2.  What is your quest?
3.  What is your favorite color?
4.  What is the capital of Assyria?
5.  What is the air-speed velocity of an unladen swallow?
```

This works for example lists, numbered lists, bullet lists, and definition
lists, using the same format, e.g.,

``` markdown
-   [q1] What is your favorite color?
```

For definition lists, the version tag goes at the beginning of the definition
item, e.g.,

``` markdown
Third Question
:   [q1] What is your favorite color?
```

Elements that accept attribute tags can be filtered by setting the `v` or
`version` attribute:

```
[A span that will only show up in q1]{v=q1}

# This is a makeup quiz {version=makeup} 
```

If no version is defined, but an output file is specified, the version is inferred from the
name of the output file, e.g.,

``` sh
pandoc quiz.md --lua-filter pandoc-questions.lua -o quiz-q3.pdf 
```

The regex for inferring the version looks for any text between the last hyphen
and the last period. This behavior can be overridden by explicitly setting the
version in the metadata (to override the inferred version without selecting
any version, use `--metadata version=nil`).

