<!-- markdownlint-disable MD013 -->

# Tmux Cheatsheet

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Tmux Cheatsheet](#tmux-cheatsheet)
  - [sessions](#sessions)
  - [windows](#windows)
  - [panes](#panes)
  - [misc](#misc)
  - [prompt](#prompt)
  - [TPM (tmux package manager)](#tpm-tmux-package-manager)
  - [credit](#credit)
<!--toc:end-->

<!-- prettier-ignore-end -->

## sessions

| command                       | action                                       |
| ----------------------------- | -------------------------------------------- |
| `tmux`                        | start new session                            |
| `tmux new -s myname`          | start new session with session name "myname" |
| `tmux a #`                    | attach to session                            |
| `tmux a -t myname`            | attach to named session with name "myname"   |
| `tmux ls`                     | list sessions                                |
| `tmux kill-session -t myname` | kill session with name "myname"              |
| `tmux kill-server`            | kill all tmux open sessions and server       |
| `<C-b>d`                      | detach session                               |

## windows

| command  | action          |
| -------- | --------------- |
| `<C-b>c` | create window   |
| `<C-b>w` | list windows    |
| `<C-b>n` | next window     |
| `<C-b>p` | previous window |
| `<C-b>f` | find window     |
| `<C-b>,` | name window     |
| `<C-b>&` | kill window     |

## panes

| command        | action                                                                       |
| -------------- | ---------------------------------------------------------------------------- |
| `<C-b>%`       | vertical split                                                               |
| `<C-b>"`       | horizontal split                                                             |
| `<C-b>q`       | show pane numbers (when numbers show up, type the number to go to that pane) |
| `<C-b>x`       | kill pane                                                                    |
| `<C-b>z`       | toggle pane zoom                                                             |
| `<C-b>{`       | move current pane left                                                       |
| `<C-b>}`       | move current pane right                                                      |
| `<C-b><space>` | toggle between layouts                                                       |

## misc

| command  | action             |
| -------- | ------------------ |
| `<C-b>t` | big clock          |
| `<C-b>?` | show shortcut help |
| `<C-b>:` | enter prompt       |
| `<C-b>[` | enter copy mode    |

## prompt

type `<C-b> :` to enter prompt mode

| command                  | action                                    |
| ------------------------ | ----------------------------------------- |
| `:source ~/.tmux.conf`   | source tmux configuration                 |
| `:swap-window -s 3 -t 1` | swap window number 3 with window number 1 |

## TPM (tmux package manager)

| command      | action            |
| ------------ | ----------------- |
| `<C-b>I`     | install plugins   |
| `<C-b>U`     | update plugins    |
| `<C-b><M-u>` | uninstall plugins |

## credit

[How do I reorder tmux windows](https://superuser.com/questions/343572/how-do-i-reorder-tmux-windows)
[tmux shortcuts & cheatsheet](https://gist.github.com/MohamedAlaa/2961058)
