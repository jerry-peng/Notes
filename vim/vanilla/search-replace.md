<!-- markdownlint-disable MD013 -->

# Vim Search and Replace

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Search and Replace](#vim-search-and-replace)
  - [Search](#search)
    - [Search char](#search-char)
    - [Search in file](#search-in-file)
    - [Search match operation](#search-match-operation)
  - [Line ranges](#line-ranges)
  - [Substitution](#substitution)
    - [Magical patterns](#magical-patterns)
    - [Regex match capture](#regex-match-capture)
    - [Command/search history](#commandsearch-history)
    - [Additional commands](#additional-commands)
    - [Substitute flags](#substitute-flags)
    - [Global command](#global-command)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Search

### Search char

| keystroke | action                                           |
| --------- | ------------------------------------------------ |
| `f[char]` | move to next char on the current line            |
| `F[char]` | move to previous char on the current line        |
| `t[char]` | move to before next char on the current line     |
| `T[char]` | move to before previous char on the current line |
| `;`       | go to next searched (f/F/t/T) character          |
| `,`       | go to previous searched (f/F/t/T) character      |

### Search in file

| keystroke  | action                                                                                                                     |
| ---------- | -------------------------------------------------------------------------------------------------------------------------- |
| `/text`    | search `text`                                                                                                              |
| `/\vtext$` | very magic search regex `text$`, which searches line ending with `text`. (See [Magical patterns](#magical-patterns) below) |
| `n`        | go to next found occurrence                                                                                                |
| `n`        | go to prev found occurrence                                                                                                |
| `*`        | Search forward the word under cursor                                                                                       |
| `#`        | Search backward the word under cursor                                                                                      |

### Search match operation

| keystroke | action                                                                                                                                                   |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gn`      | highlight next search, can be used as a motion, e.g. `/search`, `cgn`, insert text, then use `.` to replace next match, instead of having to enter `n.`. |
| `gN`      | similar to `gn`, but jumps to previous search                                                                                                            |

## Line ranges

| keystroke | action                                        |
| --------- | --------------------------------------------- |
| `.`       | current line                                  |
| `$`       | last line of current buffer                   |
| `%`       | entire file (same as `1,$`)                   |
| `*`       | last visual mode selection                    |
| `'<`      | beginning position of selected in visual mode |
| `'>`      | end position of selected in visual mode       |

Other examples:

| keystroke | action                                                   |
| --------- | -------------------------------------------------------- |
| `:1,40d`  | delete line 1 to 40                                      |
| `:2,$d`   | delete line 2 to end of file                             |
| `:.,$d`   | delete current line to end of file                       |
| `:%d`     | delete every line                                        |
| `:'<,$d`  | delete first line selected in visual mode to end of file |

## Substitution

Search command: `:s/<pattern>/<replacement>/<flags>`, which replace pattern with replacement text, and flags modifies command behavior. Replacement text is not required.

The `/` separator can be any characters except:

- alphanumerical characters (`[a-z][A-Z][0-9]`)
- double quote `"`
- pipe "|"

| keystroke                       | action                                                                                                                                |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `:s/pattern/replacement/`       | substitute first occurrence of `pattern` on current line with `replacement`                                                           |
| `:s#pattern#replacement#`       | equivalent substitution to the command above, useful if there are URL in pattern or replacement                                       |
| `:s/pattern/`                   | delete first occurrence of `pattern` on the current line                                                                              |
| `:s/pattern/replacement/g`      | substitute every occurrence of `pattern` on the current line                                                                          |
| `:%s/pattern/replacement/`      | substitute every first occurrence of `pattern` on each line                                                                           |
| `:%s/pattern/replacement/g`     | substitute every occurrence of `pattern` on each line                                                                                 |
| `:1,10s/pattern/replacement/`   | substitute every first occurrence of `pattern` on first 10 lines                                                                      |
| `:s/pattern/replacement/ 10`    | substitute every first occurrence of `pattern` for current and next 10 lines                                                          |
| `:1,10s/pattern/replacement/ 5` | substitute every first occurrence of `pattern` for 10 lines and 5 lines below last line of range                                      |
| `:s`                            | repeat last substitute command without flags                                                                                          |
| `:s g 10`                       | repeat last substitution without its flag, and add a new flag `g`, substitution happens 10 lines after last line of last substitution |

To only substitute highlighted texts, enter visual mode, then enter `:` to enter command mode. The command would be prefixed with visual mode position ranges, which is `:'<,'>`, so the substitution command only runs in highlighted texts. Other commands can be run as well.

### Magical patterns

| keystroke | action                                                                             |
| --------- | ---------------------------------------------------------------------------------- |
| `\v`      | very magic, have access to all regex metacharacters                                |
| `\V`      | very nomagic, have access to no regex metacharacter                                |
| `:sm`     | substitute magic, have access to all regex metacharacters except `(`, `)` and `\|` |
| `:sno`    | substitute nomagic, have access to no regex metacharacter except `$`               |

For example, following commands are equivalent:

- `:%s/\V(/`
- `:%sm/(/`
- `:%s/\(/`

Metacharacter `~` represents the latest substituted string

### Regex match capture

Vim substitution supports capturing regex matches. Regex match can be captured inside escaped parenthesis `\(\)`, and each capture can be referenced sequentially with `\1`, `\2`, and so on.

E.g. for following conversion:

```text
// convert from:
include{"aaa"}
include{"bbb"}
include{"ccc"}

// to:
"aaa",
"bbb",
"ccc",
```

### Command/search history

| keystroke    | action                                               |
| ------------ | ---------------------------------------------------- |
| `q:`         | open command line history                            |
| `q/` or `q?` | open search history                                  |
| `<C-f>`      | open command line history while in command line mode |

### Additional commands

| keystroke | action                                                                                                         |
| --------- | -------------------------------------------------------------------------------------------------------------- |
| `:&&`     | repeat last substitute with its flags                                                                          |
| `:~`      | repeat last substitute command with same replacement, but with last used search pattern.                       |
| `&`       | in normal mode, repeat last substitute without range and flags                                                 |
| `g&`      | in normal mode, repeat last substitute with range and flags and replace pattern with last used search pattern. |

### Substitute flags

| keystroke | action                                         |
| --------- | ---------------------------------------------- |
| `&`       | use flags from previous substitute commands    |
| `c`       | ask to confirm each substitution               |
| `g`       | replace all occurrences in each line           |
| `i`       | pattern is case-insensitive                    |
| `I`       | pattern is case-sensitive                      |
| `n`       | only report number of match without substitute |

### Global command

| keystroke            | action                   |
| -------------------- | ------------------------ |
| `:g/pattern/command` | runs command on patterns |

Example:

| keystroke    | action                                        |
| ------------ | --------------------------------------------- |
| :g/useless/d | delete all lines containing pattern `useless` |
