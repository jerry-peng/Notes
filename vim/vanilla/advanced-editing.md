<!-- markdownlint-disable MD013 -->

# Vim Advanced editing

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Advanced editing](#vim-advanced-editing)
  - [Source ".vimrc" file](#source-vimrc-file)
  - [Modify letter casing](#modify-letter-casing)
  - [Replace char](#replace-char)
  - [Joining/formatting texts](#joiningformatting-texts)
  - [Manipulating numbers](#manipulating-numbers)
  - [Sorting text](#sorting-text)
  - [Edit file](#edit-file)
  - [Filepath under cursor](#filepath-under-cursor)
  - [Return to mode](#return-to-mode)
  - [Normal mode commands](#normal-mode-commands)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Source ".vimrc" file

| keystroke                  | action                                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------------------------- |
| `:source <path to .vimrc>` | Source `.vimrc` file                                                                                     |
| `:source %`                | Source current file in buffer, useful when editing `.vimrc` file. `%` is shorthand for current file path |

## Modify letter casing

Works in visual mode.

| command | action                                |
| ------- | ------------------------------------- |
| `~`     | toggle case of character under cursor |
| `g~`    | toggle case of text object            |
| `gu`    | to lowercase of text object           |
| `gU`    | to uppercase of text object           |

## Replace char

| command   | action                                                                                                             |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| `r[char]` | replace a single char with specified char, or in visual mode, replaces each highlighted text with input character. |
| `R`       | enter replace mode to replace a block of chars                                                                     |

## Joining/formatting texts

| command | action                                    |
| ------- | ----------------------------------------- |
| `J`     | join line below to current line           |
| `gq`    | format to textwidth, works in visual mode |

## Manipulating numbers

| keystroke | action                                                                                                                                          |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `<C-a>`   | increase the first digit or number on the line                                                                                                  |
| `<C-x>`   | decrease the first digit or number on the line                                                                                                  |
| `g<C-a>`  | first number of each line will be incremented sequentially if multiple lines are selected (useful for numbered list), otherwise same as `<C-a>` |
| `g<C-x>`  | first number of each line will be decremented sequentially if multiple lines are selected (useful for numbered list), otherwise same as `<C-x>` |

A prefix count can be prepended, such as `12<C-a>`.

Note: these keystrokes can work on unsigned binary, octal, hexadecimal numbers, and alphabetical characters. Behaviors depend on value of option `nrformats`. The option should not have `alpha` as part values, or keystrokes will increment/decrement the first alphabetical character of the line.

## Sorting text

| keystroke           | action                                                                 |
| ------------------- | ---------------------------------------------------------------------- |
| `:sort` or `:sor`   | sort lines depending on range, if no range given, all lines are sorted |
| `:sort!` or `:sor!` | reverse order of `:sort`                                               |

Options:

| keystroke   | action                                                        |
| ----------- | ------------------------------------------------------------- |
| `i`         | ignore case                                                   |
| `n`         | sort depending on the first decimal on the line               |
| `f`         | sort depending on the first float on the line                 |
| `/pattern/` | sort depending on what comes after the match                  |
| `r`         | combined with `/pattern/`, sort depending on matching pattern |

For example, if you want to sort lines in a csv file (delimited by `,`) depending on second columns: `:sort /[^,]*,/`

## Edit file

| keystroke   | action           |
| ----------- | ---------------- |
| `:e <path>` | edit (open) file |

## Filepath under cursor

| keystroke | action                                                                                |
| --------- | ------------------------------------------------------------------------------------- |
| `gf`      | edit file located at the filepath under cursor                                        |
| `gx`      | open the file located at the filepath under cursor using system's default application |

## Return to mode

| keystroke               | action                                                         |
| ----------------------- | -------------------------------------------------------------- |
| `gi` (TODO, overridden) | move to last insertion location and switch to insert mode      |
| `gv`                    | start visual mode and highlight previous visual mode selection |

## Normal mode commands

| keystroke   | action                                                                 |
| ----------- | ---------------------------------------------------------------------- |
| `:norm`     | apply normal mode keystroke in command mode                            |
| `:norm!`    | apply normal mode keystroke in command mode using default vim mapping. |
| `:norm daw` | delete word under cursor                                               |

Example:

| keystroke             | action                                            |
| --------------------- | ------------------------------------------------- |
| `:g/useless/norm gu$` | lowercase every line containing pattern `useless` |
