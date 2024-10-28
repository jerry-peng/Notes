<!-- markdownlint-disable MD013 -->

# Vim Marks

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Marks](#vim-marks)
  - [Mark navigation](#mark-navigation)
  - [Special marks](#special-marks)
<!--toc:end-->

<!-- prettier-ignore-end -->

Marks `a-z` are local to one buffer, and marks `A-Z` are global to multiple buffers.

If you have a `viminfo` file set via option `viminfo` in vim, or if `shada` file set via option `shada` in neovim, marks are persisted.

Marks `0-9` are available when using a `viminfo` file in vim and `shada` file in neovim. They store position of your cursor each time you quit a file.
Mark `0` stores the last position, and mark `1` stores position before the last one, and so on.

| keystroke                         | action                                                                                                           |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Create mark**                   |                                                                                                                  |
| `m[letter]`                       | mark current cursor location                                                                                     |
| **Jump to mark**                  |                                                                                                                  |
| `'<mark>`                         | jump to first non-blank character of the line where mark was set, can be used as motion                          |
| `` `<mark> ``                     | jump to exact position where mark was set, can be used as motion                                                 |
| `g'<mark>`                        | same as `'<mark>` without changing the jump list                                                                 |
| g\`\<mark\>                       | same as `` `<mark> `` without changing the jump list                                                             |
| **Display mark**                  |                                                                                                                  |
| `:marks`                          | display mark set                                                                                                 |
| `:marks <mark>`                   | display some specific marks                                                                                      |
| `:marks <>`                       | display marks `<` and `>`                                                                                        |
| **Clear marks**                   |                                                                                                                  |
| `:delmarks <mark>`                | delete mark                                                                                                      |
| `:delmarks!` or `:delm!`          | delete all marks in range `a-z`                                                                                  |
| `` `a,`bs/pattern/replacement/ `` | substitute first match of pattert with replacement from exact position of mark `a` to exact position of mark `b` |

## Mark navigation

| keystroke | action                                                    |
| --------- | --------------------------------------------------------- |
| `]'`      | jump to first character of next line with lowercase mark  |
| `['`      | jump to first character previous line with lowercase mark |
| `` ]` ``  | jump to first character of next line with lowercase mark  |
| `` [` ``  | jump to first character previous line with lowercase mark |

## Special marks

| keystroke            | action                                                                                           |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `` `. ``             | jump to position where last changed occurred in current buffer                                   |
| `` `" ``             | jump to position where last exited current buffer                                                |
| `` `[0-9] ``         | jump to position in last nth file edited                                                         |
| `''`                 | jump back start of line in current buffer where jumped from                                      |
| ` `` `               | jump to position in current buffer where jumped from (or where you've set with `m'` or `` m` ``) |
| `` `[ `` or `` `] `` | jump to the first or last character of previously changed/deleted/yanked content                 |
| `m<` or `m>`         | set marks `'<` or `'>`, can be handy for keystroke `gv`                                          |
| `` `< `` or `` `> `` | jump to beginning or end of last visual selection                                                |
