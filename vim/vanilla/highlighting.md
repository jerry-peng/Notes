# highlighting

| command          | action                                                |
| ---------------- | ----------------------------------------------------- |
| `v`              | start visual mode, move cursor to highlight           |
| `V`              | start linewise visual mode, move cursor to highlight  |
| `<C-v>`          | start visual block mode, move cursor to highlight     |
| `v[motion]`      | start visual mode, highlight according to motion      |
| `v[text object]` | start visual mode, highlight according to text object |
| `gv`             | reselect                                              |
| `Esc`            | exit visual mode                                      |

## Visual mode commands

| command | action                           |
| ------- | -------------------------------- |
| `O`     | move to corner of block          |
| `o`     | move to other end of marked area |

To insert text on same column on multiple lines (useful for commenting code):

1. `<C-v>` to enter visual block mode
1. Use `j`/`k` to select lines
1. Use `I` (capitalized) to enter insert mode
1. Insert text
1. Press Esc, and text should be inserted in lines highlighted at the same column
