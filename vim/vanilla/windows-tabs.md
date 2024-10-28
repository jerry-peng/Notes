<!-- markdownlint-disable MD013 -->

# Windows and Tabs

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Windows and Tabs](#windows-and-tabs)
  - [Windows](#windows)
    - [Scroll window](#scroll-window)
  - [Tabs](#tabs)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Windows

| keystroke                     | action                                                      |
| ----------------------------- | ----------------------------------------------------------- |
| `:new`                        | create a new window                                         |
| `<C-w>[<Arrow Keys>/h/j/k/l]` | move between windows.                                       |
| `<C-w>w`                      | move to next window                                         |
| `<C-w>q`                      | quit current window                                         |
| **split window**              |                                                             |
| `<C-w>s` or `sp`              | split window horizontally                                   |
| `<C-w>v` or `vsp`             | split window vertically                                     |
| `<C-w>n`                      | split window horizontally and edit a new file               |
| `<C-w>^`                      | split window horizontally and edit file in alternate buffer |
| `<Buffer ID><C-w>^`           | split window horizontally and edit file in buffer ID        |
| **move/swap window**          |                                                             |
| `<C-w>T`                      | move current window to a new tab                            |
| `<C-w>r`                      | rotate window positions from left to right                  |
| `<C-w>R`                      | rotate window positions from right to left                  |
| `<C-w>x`                      | swap position with next window                              |
| `<C-w>H`                      | move current window to the far left                         |
| `<C-w>J`                      | move current window to the far bottom                       |
| `<C-w>K`                      | move current window to the far top                          |
| `<C-w>L`                      | move current window to the far right                        |
| **resize window**             |                                                             |
| `<C-w>=`                      | resize windows to fit the screen.                           |
| `<C-w>-`                      | decrease window height                                      |
| `<C-w>+`                      | increase window height                                      |
| `<C-w><`                      | decrease window width                                       |
| `<C-w>>`                      | increase window width                                       |

### Scroll window

| keystroke | action                            |
| --------- | --------------------------------- |
| `<C-b>`   | scroll down a page                |
| `<C-f>`   | scroll up a page                  |
| `<C-d>`   | scroll down half a page           |
| `<C-u>`   | scroll up half a page             |
| `<C-e>`   | scroll down                       |
| `<C-y>`   | scroll up                         |
| `zt`      | scroll cursor to top of window    |
| `zb`      | scroll cursor to bottom of window |
| `zz`      | scroll cursor to middle of window |

## Tabs

| keystroke              | action                              |
| ---------------------- | ----------------------------------- |
| `:tabnew` or `:tabe`   | open a new tab                      |
| `:tabclose` or `:tabc` | close tab                           |
| `:tabonly` or `:tabo`  | close other tabs except current tab |
| `gt`                   | go to next tab                      |
| `gT`                   | go to previous tab                  |
