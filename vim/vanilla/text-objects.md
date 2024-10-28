<!-- markdownlint-disable MD013 -->

# Text objects

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Text objects](#text-objects)
  - [Objects](#objects)
<!--toc:end-->

<!-- prettier-ignore-end -->

Text objects can only be used in visual mode or after an operator (`d`/`c`/`y`).

The commands either start with `a`, "an" object, or `i`, "inner" object, followed by an objects.

| keystroke   | action                                                                          |
| ----------- | ------------------------------------------------------------------------------- |
| `a[object]` | include surrounding [object character/white space/empty line] of the object     |
| `i[object]` | not include surrounding [object character/white space/empty line] of the object |

## Objects

| keystroke  | action                              |
| ---------- | ----------------------------------- |
| `w`        | word                                |
| `s`        | sentence                            |
| `p`        | paragraph                           |
| `"`        | a pair of double quote              |
| `'`        | a pair of single quote              |
| `` ` ``    | a pair of back quote                |
| `(`/`)`    | parenthesized block                 |
| `[`/`]`    | parenthesized block                 |
| `{`/`}`    | parenthesized block                 |
| `<` or `>` | tag block                           |
| `t`        | a pair of tag, e.g. `<body></body>` |
