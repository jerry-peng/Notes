<!-- markdownlint-disable MD013 -->

# Vim Lists

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Lists](#vim-lists)
  - [Jump list](#jump-list)
  - [Change list](#change-list)
  - [Arglist](#arglist)
  - [Buffers](#buffers)
  - [Quickfix list](#quickfix-list)
  - [Location list](#location-list)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Jump list

| keystroke | action                         |
| --------- | ------------------------------ |
| `<C-o>`   | Go to previous cursor position |
| `<C-i>`   | Go to next cursor position     |

| `:jumps` | show jump list |

## Change list

| keystroke  | action                      |
| ---------- | --------------------------- |
| `g;`       | jump to the next change     |
| `g,`       | jump to the previous change |
| `:changes` | show change list            |

## Arglist

Argument list is a subset of buffer list, files opened with `vim` command are added automatically to argument list.
For example, running `vim file1 file2` adds `file1` and `file2` to argument list, which is also in the buffer list.

| keystroke          | action                                   |
| ------------------ | ---------------------------------------- |
| `:args`            | show arglist                             |
| `:next`            | move to next file in arglist             |
| `:prev`            | move to previous file in arglist         |
| `:first`           | move to first file in arglist            |
| `:last`            | move to last file in arglist             |
| `:argadd`          | add file to arglist                      |
| `:argdo <command>` | execute command on every file in arglist |

## Buffers

A buffer can have 3 different states:

- active - buffer is displayed in the window
- hidden - buffer is not displayed but the file is open
- inactive - buffer not displayed in empty, not linked to any file

| keystroke             | action                                                                                  |
| --------------------- | --------------------------------------------------------------------------------------- |
| `:buffers`            | show buffer list                                                                        |
| `:buffers!` or `:ls!` | show buffer list including unlisted buffer, which is indicated with synbol `u` after ID |

Each line in buffer list contains:

- buffer unique ID
- buffer state (`a` for active, `h` for hidden, `<Space>` for inactive)
- buffer name, can be filepath
- Line number where cursor is

| keystroke               | action                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------ |
| **move to buffer**      |                                                                                            |
| `:buffer <ID or Name>`  | move to buffer using ID or name                                                            |
| `:bnext` or `:bn`       | move to next buffer                                                                        |
| `:bprevious` or `:bp`   | move to previous buffer                                                                    |
| `:bfirst` or `:bf`      | move to first buffer                                                                       |
| `:blast` or `:bl`       | move to last buffer                                                                        |
| `<C-6>`                 | Switch to the alternate buffer, which is indicated in buffer list with symbol `#`          |
| `<Buffer ID><C-6>`      | Switch to a specific buffer with ID `<ID>`                                                 |
| **run command**         |                                                                                            |
| `:bufdo <command>`      | apply a command to all buffers                                                             |
| **edit buffer list**    |                                                                                            |
| `:badd <filename>`      | add `<filename>` to the buffer list                                                        |
| `:bdelete <ID or Name>` | delete a buffer by ID or name, can specify more than one ID or name delimited by `<Space>` |
| `:1,10bdelete`          | delete buffer from ID 1 to 10 inclusive                                                    |
| `%bdelete`              | delete all buffers                                                                         |

## Quickfix list

Commands such as `:vimgrep`, `:make`, `:grep` populate the quickfix list automatically

| keystroke                             | action                                                          |
| ------------------------------------- | --------------------------------------------------------------- |
| `:clist` or `:cl`                     | show quickfix list, can be used with a range as argument        |
| `:copen` or `:cope`                   | open a window (with a special buffer) to show the quickfix list |
| **move to entry in quickfix list**    |                                                                 |
| `:cc <number>`                        | move to nth entry of quickfix list                              |
| `:cnext` or `:cn`                     | move to next entry of quickfix list                             |
| `:cprevious` or `:cp`                 | move to previous entry of quickfix list                         |
| `:cfirst` or `:cfir`                  | move to first entry of quickfix list                            |
| `:clast` or `:clas`                   | move to last entry of quickfix list                             |
| **run command**                       |                                                                 |
| `:cdo <command>`                      | execute command on each entry of quickfix list                  |
| **populate quickfix list**            |                                                                 |
| `:cexpr <expr>` or `:cex <expr>`      | create quickfix list using expression result                    |
| `:caddexpr <expr>` or `:cadde <expr>` | append result of expression result to quickfix list             |

Examples:

| keystroke           | action                                         |
| ------------------- | ---------------------------------------------- |
| `:cex []`           | empty quickfix list                            |
| `:cex system("ls")` | populate quickfix list with shell command `ls` |

## Location list

Location list is similar to quickfix list, except location list is global and quickfix list is local to a window.

Commands such as `:lvimgrep`, `:lmake` populate location list automatically.

Location lists commands are similar to quickfix list commands, by replacing the initial `c` to `l`.

| keystroke          | action                                                   |
| ------------------ | -------------------------------------------------------- |
| `:llist` or `:lli` | show location list, can be used with a range as argument |
| `:ll <number>`     | move to nth entry of location list                       |
| `:lnext` or `:lne` | move to next entry of location list                      |

and so on...
