<!-- markdownlint-disable MD013 -->

# Vim Key mappings

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Key mappings](#vim-key-mappings)
  - [Symbols](#symbols)
  - [Commands](#commands)
    - [mapping characters](#mapping-characters)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Symbols

| map mode symbol | map mode                                        |
| --------------- | ----------------------------------------------- |
| no symbol       | normal/visual/select mode operator-pending mode |
| `n`             | normal mode                                     |
| `v`             | visual/select mode                              |
| `x`             | visual mode                                     |
| `s`             | select mode                                     |
| `o`             | operator-pending mode                           |
|                 |                                                 |
| `!`             | insert/command mode                             |
| `i`             | insert mode                                     |
| `c`             | command mode                                    |
| `l`             | insert/command mode and lang-arg                |

Lang-arg refers to argument of commands that accept a text character, e.g. `f` (find char) and `r` (replace char).

## Commands

Insert/command mode symbol `!` is appended after command rather than prepended.

| commands                     | description                                                       |
| ---------------------------- | ----------------------------------------------------------------- |
| `:[mode]map[!]`              | show all key mappings in specified mode                           |
| `:[mode]map[!] {lhs} {rhs}`  | map key sequence `{lhs}` to `{rhs}`                               |
| `:[mode]noremap {lhs} {rhs}` | map key sequence `{lhs}` to {rhs}` without recursive mapping      |
| `:[mode]unmap[!] {lhs}`      | remove mapping of `{lhs}` for the modes where map command applies |
| `:[mode]mapclear[!]`         | remove all mappings for specified mode                            |

### mapping characters

| keyname   | key                     |
| --------- | ----------------------- |
| `<space>` | `Space`                 |
| `<c-w>`   | `Ctrl-w`                |
| `<cr>`    | carriage return (Enter) |

Leader key can be utilized in key mappings. First define leader key: `:let mapleader = "<space>"`, then use leader key in mapping: `nnoremap <leader>an :next<cr>`.
