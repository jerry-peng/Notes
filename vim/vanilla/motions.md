<!-- markdownlint-disable MD013 -->

# Motions

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Motions](#motions)
  - [Directions](#directions)
  - [Word-wise motion](#word-wise-motion)
  - [Within line](#within-line)
  - [Within file](#within-file)
  - [Other motions](#other-motions)
  - [Window position](#window-position)
<!--toc:end-->

<!-- prettier-ignore-end -->

- Motions can be used in visual mode
- Operators that can be combined with motions or [text objects](text-objects.md).
- Most motions can be repeated a number of times by prepending the number before the motions. This also applies when combined with operators.

Examples:

| keystroke | action                                               |
| --------- | ---------------------------------------------------- |
| `ggdG`    | moves to start o f file and delete until end of file |
| `6j`      | moves cursor down 6 rows                             |
| `d6w`     | deletes 6 words                                      |

## Directions

| keystroke | action            |
| --------- | ----------------- |
| `h`       | move cursor left  |
| `j`       | move cursor down  |
| `k`       | move cursor up    |
| `l`       | move cursor right |

## Word-wise motion

| keystroke | action                                  |
| --------- | --------------------------------------- |
| `w`       | to next start of words/punctuations     |
| `W`       | to next start of words                  |
| `b`       | to previous start of words/punctuations |
| `B`       | to previous start of words              |
| `e`       | to next end of words/punctuations       |
| `E`       | to next end of words                    |
| `ge`      | to previous end of words/punctuations   |
| `gE`      | to previous end of words                |

## Within line

| keystroke | action                               |
| --------- | ------------------------------------ |
| `$`       | to end of line                       |
| `0`       | to start of line                     |
| `^`       | to first non-blank character of line |

## Within file

| keystroke       | action                   |
| --------------- | ------------------------ |
| `gg`            | top of file              |
| `G`             | bottom of file           |
| `:[num]<enter>` | to specified line number |

## Other motions

| keystroke | action                                   |
| --------- | ---------------------------------------- |
| `(`       | to previous sentence                     |
| `)`       | to next sentence                         |
| `{`       | to previous paragraph                    |
| `}`       | to next paragraph                        |
| `[m`      | move to previous method brace `{` or `}` |
| `]m`      | move to next method brace `{` or `}`     |

## Window position

| keystroke | action                   |
| --------- | ------------------------ |
| `H`       | move to top of window    |
| `M`       | move to middle of window |
| `L`       | move to bottom of window |
