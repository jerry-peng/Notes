<!-- markdownlint-disable MD013 -->

# Vim Basics

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Basics](#vim-basics)
  - [Modes](#modes)
  - [Basic Commands](#basic-commands)
    - [Basic mode change](#basic-mode-change)
    - [Help](#help)
    - [Exit/Open files](#exitopen-files)
      - [Example](#example)
  - [Editing](#editing)
    - [Insert](#insert)
    - [Undo and Redo](#undo-and-redo)
    - [Basic Operators](#basic-operators)
    - [Line-wise operation](#line-wise-operation)
    - [Delete char](#delete-char)
    - [Paste](#paste)
    - [Repeat](#repeat)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Modes

| mode        | purpose                                   |
| ----------- | ----------------------------------------- |
| normal mode | navigate the structure of the file        |
| insert mode | edit the file                             |
| visual mode | highlight portions of the file to operate |
| ex mode     | command mode                              |

- Most of the commands works in normal mode.
- Command starting with `:` runs in command mode.

## Basic Commands

### Basic mode change

| keystroke | action                                         |
| --------- | ---------------------------------------------- |
| `<Esc>`   | switch to normal mode from other modes         |
| `i`       | switch to insert mode from normal mode         |
| `v`       | switch to visual mode from normal mode         |
| `:`       | switch to command mode from normal/visual mode |

### Help

| keystroke | action        |
| --------- | ------------- |
| `:help`   | open vim help |

### Exit/Open files

| keystroke     | action                                     |
| ------------- | ------------------------------------------ |
| `:q`          | quit current window                        |
| `:w`          | write (save) current file                  |
| `:wq` or `:x` | write current file and quit current window |

- Appending `a` executes command on all files
- Appending `!` after `:q` quits without saving

#### Example

| keystroke | action                                |
| --------- | ------------------------------------- |
| `:qa!`    | quit all files/windows without saving |

## Editing

### Insert

These normal mode commands switch to insert mode

| keystroke | action                                                                             |
| --------- | ---------------------------------------------------------------------------------- |
| `i`       | insert at cursor                                                                   |
| `I`       | insert at start of line, works in visual mode, see [highlighting](highlighting.md) |
| `a`       | append after the cursor                                                            |
| `A`       | append at the end of line                                                          |
| `o`       | open blank line below current line                                                 |
| `O`       | open blank line above current line                                                 |
| `gI`      | insert text at beginning of line                                                   |

### Undo and Redo

| keystroke | action |
| --------- | ------ |
| `u`       | undo   |
| `<C-r>`   | redo   |

### Basic Operators

- In visual mode, operators operates immediately on highlighted text/line.
- In normal mode, operators operates on text selected by motion/
- Repeated operators will operate on current line.

| keystroke                    | action                                                     |
| ---------------------------- | ---------------------------------------------------------- |
| **delete/change/copy**       |                                                            |
| `d`                          | delete                                                     |
| `c`                          | change (delete and go to insert mode)                      |
| `y`                          | yank (copy)                                                |
| `dd`                         | delete line                                                |
| `cc`                         | change line (delete line and go to insert mode)            |
| `yy`                         | yank (copy) line                                           |
|                              |                                                            |
| **Indent/unindent/reformat** |                                                            |
| `>`                          | indent operator                                            |
| `<`                          | unindent operator                                          |
| `=`                          | indentation reformat operator                              |
| `>>`                         | indent current line, or highlighted lines                  |
| `<<`                         | unindent current line, or highlighted lines                |
| `<<`                         | reformat indentation of current line, or highlighted lines |

### Line-wise operation

Repeating operators will operate on cursor line.

| keystroke | action                                                 |
| --------- | ------------------------------------------------------ |
| `D`       | short hand for `d$`, delete from cursor to end of line |
| `C`       | short hand for `c$`, change from cursor to end of line |
| `Y`       | short hand for `y$`, yank from cursor to end of line   |
| `S`       | clear current line and enter insert mode               |

### Delete char

| keystroke | action                                    |
| --------- | ----------------------------------------- |
| `x`       | delete current char                       |
| `X`       | delete previous char                      |
| `s`       | delete current char and enter insert mode |

### Paste

**Note:** Paste commands paste texts stored in unnamed `"` register, which stores the most recent yanked/deleted/replaced text. For more info, visit [registers](registers.md)

| keystroke | action                    |
| --------- | ------------------------- |
| `p`       | put (paste) after cursor  |
| `P`       | put (paste) before cursor |

### Repeat

| keystroke | action               |
| --------- | -------------------- |
| `.`       | repeat last commands |
