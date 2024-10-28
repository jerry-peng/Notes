<!-- markdownlint-disable MD013 -->

# vim-fugitive

<!-- prettier-ignore-start -->

<!--toc:start-->
- [vim-fugitive](#vim-fugitive)
  - [Basic commands](#basic-commands)
  - [Interactive git status](#interactive-git-status)
  - [Diff](#diff)
    - [file reconciliation](#file-reconciliation)
    - [piecemeal reconciliation](#piecemeal-reconciliation)
  - [Resolving merge conflict](#resolving-merge-conflict)
    - [buffspec](#buffspec)
    - [resolve with `diffget`](#resolve-with-diffget)
    - [resolve with `diffput`](#resolve-with-diffput)
    - [shorthand](#shorthand)
    - [other useful commands](#other-useful-commands)
  - [Git object database](#git-object-database)
    - [Reading from branch](#reading-from-branch)
    - [explore git object database](#explore-git-object-database)
      - [commit objects](#commit-objects)
      - [Tree objects](#tree-objects)
  - [Open GitHub URL](#open-github-url)
  - [Past commits/files](#past-commitsfiles)
    - [browse past revisions of a file](#browse-past-revisions-of-a-file)
    - [browse past commits](#browse-past-commits)
  - [Search](#search)
    - [search for text in files](#search-for-text-in-files)
    - [search for text](#search-for-text)
    - [search for text added or removed by a commit](#search-for-text-added-or-removed-by-a-commit)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Basic commands

`:Git` or `:G` for running invoking git within vim. E.g. `:G commit` runs `git commit`.

There are other fugitive wrapper commands that handles buffers upon git invocations.

| commands               | description                                                                                                            |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `:Gwrite`              | Runs `:Git add %` which stages current file                                                                            |
| `:Gread`               | Runs `:Git checkout %` which reverts to current file to last staged version                                            |
| `:GRemove`             | Runs `:Git rm %`, which deletes current file from repository, and also deletes file buffer                             |
| `:GMove dest/path`     | Runs `:Git mv % dest/path`, which moves file source to destination path, and recreates buffer under destination path   |
| `:Git`                 | Runs interactive `:Git status`. For commands, see [interactive git status](#interactive-git-status)                    |
| `:Gedit`               | Read index (most recently staged or committed) version of the file in current buffer, same as `Gedit :0` or `Gedit :%` |
| `:Gedit :path/to/file` | Read index version of file specified by path to file                                                                   |
| `:Gdiff`               | Compare working copy with index (staged) version                                                                       |

## Interactive git status

| Keystrokes | Description                                                               |
| ---------- | ------------------------------------------------------------------------- |
| `<C-n>`    | jump to next file                                                         |
| `<C-p>`    | jump to previous file                                                     |
| `-`        | add/reset file, works in visual mode                                      |
| `<Enter>`  | open current file in window below                                         |
| `=`        | inline diff for current file                                              |
| `I`        | run `git add -patch` for current file, which runs interactive stage hunks |
| `c`        | run `:Git commit`                                                         |

## Diff

`:Gdiff` runs vimdiff on working file against its index (staged) version. It opens 2 windows split vertically, left window shows index file, and right window shows working file.

### file reconciliation

| commands  | file         | description                      |
| --------- | ------------ | -------------------------------- |
| `:Gread`  | index file   | stage working file to index file |
| `:Gread`  | working file | checkout index file              |
| `:Gwrite` | index file   | checkout index file              |
| `:Gwrite` | working file | stage working file to index file |

### piecemeal reconciliation

| commands   | file         | description                   |
| ---------- | ------------ | ----------------------------- |
| `:diffget` | index file   | stage hunk to index file      |
| `:diffget` | working file | checkout hunk from index file |
| `:diffput` | index file   | checkout hunk from index file |
| `:diffput` | working file | stage hunk to index file      |

## Resolving merge conflict

To resolve merge conflict run either `Ghdiffsplit!` or `Gvdiffsplit!` to initiate 3-way horizontal or vertical merge window splits.

When using `:Gdiff` on a conflicted file, 3 windows are opened in order:

| window | buffspec | description            |
| ------ | -------- | ---------------------- |
| left   | `//2`    | target branch (`HEAD`) |
| middle | `//1`    | working copy           |
| right  | `//3`    | merge branch           |

### buffspec

`buffspec` can either be a the buffer number, or a partial match for buffer's name.

In creating buffers for the target/merge version of the file when resolving conflict, working copy file path always contain `//1`, target file path always contains `//2`, and merge file path always contain `//3`.

### resolve with `diffget`

Running `diffget` in working copy window pulls a change from one of the other buffers. Because of 3-way merge,

| commands       | description                    |
| -------------- | ------------------------------ |
| `:diffget //2` | fetch hunk from target version |
| `:diffget //3` | fetch hunk from merge version  |

### resolve with `diffput`

Running `diffput` in target or merge version will push change to working copy. No argument is needed since as `diffput` will always push change to working copy.

### shorthand

| commands   | normal mode keystrokes |
| ---------- | ---------------------- |
| `:diffget` | `do`                   |
| `:diffput` | `dp`                   |

`do` does not take arguments, so it would not work in a 3-way merge, but `dp` works.

### other useful commands

| commands  | description                              |
| --------- | ---------------------------------------- |
| `[c`      | jump to previous hunk                    |
| `]c`      | jump to next hunk                        |
| `:only`   | close all windows apart from current one |
| `Gwrite!` | write current file to index              |

To leave vimdiff mode, run `:only` on working copy.

When calling `:Gwrite!` from vimdiff mode, it writes the current file to index and exists vimdiff.

## Git object database

`Gedit` allows opening files in other branches, and browse any git object (tags, commits, trees).

### Reading from branch

`Gedit branch:path/to/file` to open read-only buffer of file in branch. Note `%` is shorthand for current file.

### explore git object database

There are 4 kinds of git object:

- blobs: corresponds to file content
- trees: corresponds to filesystem, represents a list of blobs and trees
- commits: references a tree and one or more parent commits
- tags: refers to a particular commit by name

Every git object has a SHA code, to explore, run `:Gedit SHA`

#### commit objects

When viewing commit objects, fugitive opens an interactive buffer, which looks like output of `git show SHA` in shell.

Press `<Enter>` to open new buffer containing another git object:

- press `<Enter>` on parent opens parent commit object
- press `<Enter>` on tree to open tree object
- press `<Enter>` on diff summary will open `:Gdiff` on the file before and after the commit

#### Tree objects

When viewing tree objects, fugitive opens a textual representation of the tree object, which looks like output of `git show SHA` in shell.

`git ls-tree SHA` shows more information in the shell. It includes the SHA code for every object referenced by that tree.

Pressing `<Enter>` on a line representing file or directory will open buffer for that object. To go to parent object, use command `:edit %:h`, or go to previous jump list location `<C-o>`.

| commands | description                                                               |
| -------- | ------------------------------------------------------------------------- |
| `a`      | switch between `git show SHA` and `git ls-tree SHA` in interactive buffer |
| `C`      | jump back to to commit object                                             |

## Open GitHub URL

`:Gbrowse` can open git object webpage on GitHub. It can recognize whether active buffer contains a blob, tree, commit or tag.

If `:Gbrowse` is triggered from visual mode in a file, then selected lines will be highlighted in GitHub page.

## Past commits/files

### browse past revisions of a file

`Glog` command allows examining previous revisions of a file. It does by loading each revision into its own buffer, and queuing them in quickfix list.

| commands                     | description                                                                         |
| ---------------------------- | ----------------------------------------------------------------------------------- |
| `:Glog`                      | load all previous revisions of current file into quickfix list                      |
| `:Glog -10`                  | load last 10 previous revisions of current file into quickfix list                  |
| `:Glog -10 --reverse`        | load last 10 previous revisions of current file into quickfix list in reverse order |
| `:Glog -1 --until=yesterday` | load the last version of current file checked in before last midnight               |

### browse past commits

| commands     | description                                                                        |
| ------------ | ---------------------------------------------------------------------------------- |
| `:Glog --`   | load all ancestral commit objects into quickfix list                               |
| `:Glog -- %` | load all ancestral commit objects that touched the current file into quickfix list |

## Search

### search for text in files

`Ggrep` is wrapper for `git grep`, and results are populated in quickfix list.

| commands                   | description                                                          |
| -------------------------- | -------------------------------------------------------------------- |
| `:Ggrep findme`            | search for 'findme' in working copy file (excluding untracked files) |
| `:Ggrep --cached findme`   | search for 'findme' in index                                         |
| `:Ggrep findme branchname` | search for 'findme' in branch 'branchname'                           |
| `:Ggrep findme tagname`    | search for 'findme' in tag 'tagname'                                 |
| `:Ggrep findme SHA`        | search for 'findme' in the commit/tag identified by SHA              |

### search for text

To search for text of commit messages, run `git log --grep=findme`.

| commands                   | description                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------- |
| `:Glog --grep=findme --`   | search for 'findme' in all ancestral commit messages                                  |
| `:Glog --grep=findme -- %` | search for 'findme' in all ancestral commit messages that touches current active file |

### search for text added or removed by a commit

Git allows for searching text that was added or removed by a commit using the pickaxe option: `git log -Sfindme`

This command goes through each commit, comparing the before and after state of each file. If specified string was added or removed, it will show up in result.

| commands             | description                                                                                      |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `Glog -Sfindme --`   | search for 'findme' in the diff for each ancestral commit                                        |
| `Glog -Sfindme -- %` | search for 'findme' in the diff for each ancestral commit that touches the currently active file |
