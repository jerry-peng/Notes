<!-- markdownlint-disable MD013 -->

# grapple.nvim

## Tags window

| key binding   | description                     |
| ------------- | ------------------------------- |
| `?`           | show help                       |
| `<CR>`        | select tag                      |
| `<c-s>`       | select tag and split horizontal |
| `\|`          | select tag and split horizontal |
| `1`,...,`9`   | select tag at an index          |
| delete a line | delete tag                      |
| move a line   | reorder tag                     |
| `<R>`         | rename tag                      |
| `<C-q>`       | send all tags to quickfix list  |
| `-`           | navigate up to scopes window    |

## Scopes window

| key binding | description                                  |
| ----------- | -------------------------------------------- |
| `?`         | show help                                    |
| `<CR>`      | open tags window for selected scope          |
| `1`,...,`9` | select tags window for scope at an index     |
| `<S-CR>`    | change current scope to the one under cursor |
| `-`         | go to loaded scopes window                   |
| `g.`        | toggle hidden scopes                         |

## Loaded scopes window

| key binding | description                                     |
| ----------- | ----------------------------------------------- |
| `?`         | show help                                       |
| `<CR>`      | open tags window for selected loaded scope      |
| `1`,...,`9` | select tags window for loaded scope at an index |
| `x`         | unload tags for scope                           |
| `X`         | reset tags for scope                            |
| `-`         | go to scopes window                             |
| `g.`        | toggle showing loaded and unloaded scopes       |

## Persistence

All tags are stored in json blobs in `~/.local/share/nvim/grapple` directory.
