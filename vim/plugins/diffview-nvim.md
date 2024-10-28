<!-- markdownlint-disable MD013 -->

# diffview.nvim

<!-- prettier-ignore-start -->

<!--toc:start-->
- [diffview.nvim](#diffviewnvim)
  - [Diffview](#diffview)
    - [Diffview Examples](#diffview-examples)
  - [Diffview File History](#diffview-file-history)
    - [File History Examples](#file-history-examples)
  - [Keymaps](#keymaps)
    - [General](#general)
    - [Diff View keymaps](#diff-view-keymaps)
    - [File Panel Keymaps](#file-panel-keymaps)
    - [File History Panel Keymaps](#file-history-panel-keymaps)
    - [Option Panel Keymaps](#option-panel-keymaps)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Diffview

`:Diffview` can be used to look at unstaged changes, or merge conflicts. Merge conflicts will trigger 3-way diff.

Command:

```vim
:DiffviewOpen [git-rev] [options] [ -- {paths...}]
```

### Diffview Examples

```vim
" Diff the working tree against the index
:DiffviewOpen"

" Diff the working tree against a specific commit
:DiffviewOpen HEAD~2
:DiffviewOpen d4a7b0d

" Diff a commit range
:DiffviewOpen HEAD~4..HEAD~2
:DiffviewOpen d4a7b0d..519b30e

" Diff the changes introduced by a specific commit (kind of like `git show d4a7b0d`)
:DiffviewOpen d4a7b0d^!

" Diff HEAD against it's merge base in origin/main
:DiffviewOpen origin/main...HEAD

" Limit the scope to the given paths
:DiffviewOpen HEAD~2 -- lua/diffview plugin

" Hide untracked files
:DiffviewOpen -uno
```

Note that `...` (symmetric difference notation) includes commits reachable from either start/end revisions but exclude not from both, while `..` (range notation) includes commits reachable from end revision but not start revision. For more info visit [Git revisions](https://git-scm.com/docs/revisions).

## Diffview File History

`:DiffviewFileHistory` opens file history view that lists all commits that affects given paths.

Command:

```vim
:[range]DiffviewFileHistory [paths] [options]

```

### File History Examples

```vim
" History for the current branch
:DiffviewFileHistory

" History for the current file
:DiffviewFileHistory %

" History for a specific file
:DiffviewFileHistory path/to/some/file.txt

" History for a specific directory
:DiffviewFileHistory path/to/some/directory

" History for multiple paths
:DiffviewFileHistory multiple/paths foo/bar baz/qux

" Compare history against a fixed base
DiffviewFileHistory --base=HEAD~4
:DiffviewFileHistory --base=LOCAL

" History for a specific rev range
:DiffviewFileHistory --range=origin..HEAD
:DiffviewFileHistory --range=feat/some-branch

" Inspect diffs for Git stashes
:DiffviewFileHistory -g --range=stash
```

## Keymaps

### General

| key binding | descriptions |
| ----------- | ------------ |
| `g?`        | show help    |

### Diff View keymaps

| key binding        | descriptions                                           |
| ------------------ | ------------------------------------------------------ |
| **select entry**   |                                                        |
| `<Tab>`            | select next entry                                      |
| `<S-Tab>`          | select previous entry                                  |
| `[F`               | select first entry                                     |
| `]F`               | select last entry                                      |
| **open file**      |                                                        |
| `gf`               | open file in previous tabpage                          |
| `<C-w><C-f>`       | open file in new split                                 |
| `<C-w>gf`          | open file in new tabpage                               |
| **file panel**     |                                                        |
| `<leader>e`        | focus on file panel                                    |
| `<leader>b`        | toggle file panel                                      |
| **cycle layout**   |                                                        |
| `g<C-x>`           | cycle through available layouts                        |
| **merge tool**     |                                                        |
| `[x`               | jump to previous conflict                              |
| `]x`               | jump to next conflict                                  |
| `<leader>co`       | choose OURS version of a conflict                      |
| `<leader>ct`       | choose THEIRS version of a conflict                    |
| `<leader>cb`       | choose BASE version of a conflict                      |
| `<leader>ca`       | choose ALL version of a conflict                       |
| `<leader>cO`       | choose OURS version for a whole file                   |
| `<leader>cT`       | choose THEIRS version for a whole file                 |
| `<leader>cB`       | choose BASE version for a whole file                   |
| `<leader>cA`       | choose ALL version for a whole file                    |
| `dX`               | delete conflict region for a whole file                |
| **diff shorthand** | Keybind is available depending on number of diff panes |
| `1do`              | obtain diff hunk from BASE version of the file         |
| `2do`              | obtain diff hunk from OURS version of the file         |
| `3do`              | obtain diff hunk from THEIRS version of the file       |

### File Panel Keymaps

| key binding      | descriptions                                     |
| ---------------- | ------------------------------------------------ |
| **git actions**  |                                                  |
| `-`/`s`          | toggle stage; stage/unstage an entry             |
| `S`              | stage all entries                                |
| `U`              | unstage all entries                              |
| `X`              | restore an entry                                 |
| `L`              | open commit log                                  |
| **fold**         |                                                  |
| `zo`             | open fold                                        |
| `zc`/`h`         | close fold                                       |
| `za`             | toggle fold                                      |
| `zR`             | open all folds                                   |
| `zM`             | close all folds                                  |
| **select entry** |                                                  |
| `<Tab>`          | select next entry                                |
| `<S-Tab>`        | select previous entry                            |
| `[F`             | select first entry                               |
| `]F`             | select last entry                                |
| **open file**    |                                                  |
| `gf`             | open file in previous tabpage                    |
| `<C-w><C-f>`     | open file in new split                           |
| `<C-w>gf`        | open file in new tabpage                         |
| **file list**    |                                                  |
| `i`              | toggle between list and tree views               |
| `f`              | toggle flatten empty subdirectories in tree view |
| `R`              | refresh file list                                |
| **file panel**   |                                                  |
| `<leader>e`      | focus on file panel                              |
| `<leader>b`      | toggle file panel                                |
| **cycle layout** |                                                  |
| `g<C-x>`         | cycle through available layouts                  |
| **merge tool**   |                                                  |
| `[x`             | jump to previous conflict                        |
| `]x`             | jump to next conflict                            |
| `<leader>cO`     | choose OURS version for a whole file             |
| `<leader>cT`     | choose THEIRS version for a whole file           |
| `<leader>cB`     | choose BASE version for a whole file             |
| `<leader>cA`     | choose ALL version for a whole file              |
| `dX`             | delete conflict region for a whole file          |

### File History Panel Keymaps

| key binding      | descriptions                          |
| ---------------- | ------------------------------------- |
| `g!`             | open options panel                    |
| `<C-A-d>`        | open entry under cursor in a diffview |
| `y`              | copy hash                             |
| `L`              | open commit log                       |
| `X`              | restore entry                         |
| **fold**         |                                       |
| `zo`             | open fold                             |
| `zc`/`h`         | close fold                            |
| `za`             | toggle fold                           |
| `zR`             | open all folds                        |
| `zM`             | close all folds                       |
| **select entry** |                                       |
| `<Tab>`          | select next entry                     |
| `<S-Tab>`        | select previous entry                 |
| `[F`             | select first entry                    |
| `]F`             | select last entry                     |
| **open file**    |                                       |
| `gf`             | open file in previous tabpage         |
| `<C-w><C-f>`     | open file in new split                |
| `<C-w>gf`        | open file in new tabpage              |
| **file panel**   |                                       |
| `<leader>e`      | focus on file panel                   |
| `<leader>b`      | toggle file panel                     |
| **cycle layout** |                                       |
| `g<C-x>`         | cycle through available layouts       |

### Option Panel Keymaps

| key binding | descriptions          |
| ----------- | --------------------- |
| `<Tab>`     | change current option |
