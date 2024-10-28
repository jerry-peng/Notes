<!-- markdownlint-disable MD013 -->

# Vim Registers

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Registers](#vim-registers)
  - [Basic register operations](#basic-register-operations)
  - [Types of registers](#types-of-registers)
  - [Using registers in insert/command line modes](#using-registers-in-insertcommand-line-modes)
  - [Expression register](#expression-register)
  - [Clearing a register](#clearing-a-register)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Basic register operations

| keystroke              | action                                       |
| ---------------------- | -------------------------------------------- |
| `:registers` or `:reg` | show contents of registers                   |
| `"<reg>`               | specifies the register to be read or written |

Examples:

| keystroke | action                                        |
| --------- | --------------------------------------------- |
| `"add`    | delete line and store content in register `a` |
| `"ap`     | paste content in register `a`                 |

## Types of registers

- The unnamed register (`"`) - contains last deleted/changed/yanked content even if one register was specified.
- The numbered registers(from `0` to `9`)

  - `0` contains content of last yank
  - `1` to `9` is a stack containing content deleted or changed

- The small delete register (`-`) - contains any deleted or changed content smaller than one line, not written if register `"` is specified.
- The named registers (`a` to `z`)

  - vim will not write to them if not specified with keystroke `"`
  - Use uppercase name of each register to append to it

- The read only registers(`.`, `%` and `:`)

  - `.` contains last inserted text
  - `%` contains the name of current file
  - `:` contains most recent command line executed

- The alternate buffer register (`#`) - contains alternate buffer for current window.
- The expression register (`=`) - stores result of an expression.
- The selection register (`+` and `*`)

  - `+` is synchronized with system clipboard
  - `*` is synchronized with selection clipboard (only on \*nix systems)

- The black hole register (`_`) - Everything written here will disappear forever.
- The last search pattern register (`/`) - contains last search

## Using registers in insert/command line modes

| keystroke    | action                                            |
| ------------ | ------------------------------------------------- |
| `<C-r><reg>` | put content of register `<reg>` in current buffer |

## Expression register

| keystroke | action                                                                                                                                                                     |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<C-r>=`  | Use expression register in insert/command line mode. This will move to command line mode, and from there, type anyt Vimscript expression such as `system("ls")` and `4+4`. |

## Clearing a register

| keystroke | action                                                        |
| --------- | ------------------------------------------------------------- |
| `qaq`     | start a recording and stop recording, which clears a register |
