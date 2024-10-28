<!-- markdownlint-disable MD013 -->

# Comment.nvim

## Normal mode

| key binding         | description                                                         |
| ------------------- | ------------------------------------------------------------------- |
| `[count]gcc`        | toggle number of line using linewise comment, count can be omitted  |
| `[count]gbc`        | toggle number of line using blockwise comment, count can be omitted |
| `gc[count]{motion}` | toggle region using linewise comment                                |
| `gb[count]{motion}` | toggle region using blockwise comment                               |
| `gbo`               | add line comment on line below                                      |
| `gbO`               | add line comment on line above                                      |
| `gbA`               | add line-comment on end of line                                     |

## Visual mode

| key binding | description                                       |
| ----------- | ------------------------------------------------- |
| `gc`        | toggle highlighted region using linewise comment  |
| `gb`        | toggle highlighted region using blockwise comment |
