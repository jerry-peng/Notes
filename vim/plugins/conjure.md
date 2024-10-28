<!-- markdownlint-disable MD013 -->

# Conjure

## Commands

```vim
" Evaluate code, works with visual selections
:ConjureEval [code]
:%ConjureEval "Evaluate current buffer

" Connect to given host and port, only works with network based REPL connections
:ConjureConnect [host] [port]
:ConjureConnect
:ConjureConnect 5678
:ConjureConnect staging.my-app.com 5678
```

## Keymaps

Most of conjure's keymaps uses local leader, which is set to `,` in my config.

| key binding    | descriptions                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| **log buffer** |                                                                                                              |
| `,ls`          | open log buffer in horizontal split                                                                          |
| `,lv`          | open log buffer in vertical split                                                                            |
| `,lt`          | open log buffer in new tab                                                                                   |
| `,le`          | open log buffer in current window, use `<C-o>` to jump back                                                  |
| `,lq`          | close all visible log windows in current tab                                                                 |
| `,lr`          | soft reset log buffer by deleting lines, buffer window is kept open                                          |
| `,lR`          | hard reset log buffer by deleting buffer and any local mappings/settings. buffer window is closed            |
| **evaluate**   |                                                                                                              |
| `,E`           | evaluate selection in visual mode                                                                            |
| `,E[motion]`   | evaluate selection under motion                                                                              |
| `,ee`          | evaluate current form under cursor                                                                           |
| `,ece`         | evaluate current form under cursor and display result as a comment at the end of line, or below current line |
| `,er`          | evaluate root form under cursor                                                                              |
| `,ecr`         | evaluate root form under cursor and display result as a comment at the end of line, or below current line    |
| `,ew`          | evaluate word under cursor                                                                                   |
| `,ecw`         | evaluate word under cursor and display result as a comment at the end of line, or below current line         |
| `,e!`          | evaluate form under cursor and replace with result                                                           |
| `,em[mark]`    | evaluate form under cursor at given mark                                                                     |
| `,ef`          | evaluate current file from disk, which may be different from what's in buffer                                |
| `,eb`          | evaluate current buffer contents, which may be different from what's on disk                                 |
| `,gd`          | go to definition of word under cursor                                                                        |
| `K`            | look up documentation for word under cursor                                                                  |
