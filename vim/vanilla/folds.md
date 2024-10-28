<!-- markdownlint-disable MD013 -->

# Vim Folding

| keystroke                          | action                                                          |
| ---------------------------------- | --------------------------------------------------------------- |
| `zf[motion]`                       | create a fold of lines selected by motion, or highlighted lines |
| **operate on fold on a cursor**    |                                                                 |
| `zc`                               | close one level of fold under cursor                            |
| `zC`                               | close all levels of folds under cursor                          |
| `zo`                               | open one level of fold under cursor                             |
| `zO`                               | open all levels of folds under cursor                           |
| `za`                               | toggle one level of fold under cursor                           |
| `zA`                               | toggle all levels of folds under cursor                         |
| `zv`                               | view cursor line                                                |
| **operate on all folds in buffer** |                                                                 |
| `zr`                               | open one level of fold in foldings across buffer                |
| `zR`                               | open all folds                                                  |
| `zm`                               | close one level of fold in foldings across buffer               |
| `zM`                               | close all folds                                                 |
| **delete fold**                    |                                                                 |
| `zd`                               | delete one level of fold under the cursor                       |
| `zD`                               | delete all level of folds under the cursor                      |
| `zE`                               | delete all folds                                                |
| **fold motions**                   |                                                                 |
| `[z`                               | move to start of current open fold                              |
| `]z`                               | move to end of current open fold                                |
| `zj`                               | move downward to start of next fold                             |
| `zk`                               | move upward to end of previous fold                             |
