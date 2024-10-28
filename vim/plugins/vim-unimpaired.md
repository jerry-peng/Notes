<!-- markdownlint-disable MD013 -->

# vim-unimpaired

## Next/previous

| Key binding | Description   |
| ----------- | ------------- |
| `[a`        | `:previous`   |
| `]a`        | `:next`       |
| `[A`        | `:first`      |
| `]A`        | `:last`       |
| `[b`        | `:bprevious`  |
| `]b`        | `:bnext`      |
| `[B`        | `:bfirst`     |
| `]B`        | `:blast`      |
| `[l`        | `:lprevious`  |
| `]l`        | `:lnext`      |
| `[L`        | `:lfirst`     |
| `]L`        | `:llast`      |
| `[<C-l>`    | `:lpfile`     |
| `]<C-l>`    | `:lnfile`     |
| `[q`        | `:cprevious`  |
| `]Q`        | `:clast`      |
| `]q`        | `:cnext`      |
| `[Q`        | `:cfirst`     |
| `[<C-q>`    | `:cpfile`     |
| `]<C-q>`    | `:cnfile`     |
| `[t`        | `:tprevious`  |
| `]t`        | `:tnext`      |
| `]T`        | `:Tlast`      |
| `[T`        | `:Tfirst`     |
| `[<C-t>`    | `:prprevious` |
| `]<C-t>`    | `:ptnext`     |

## File operations

| Key binding | Description                                                                                          |
| ----------- | ---------------------------------------------------------------------------------------------------- |
| `[f`        | Go to previous file alphabetically in current file's directory. In quickfix, equivalent to `:colder` |
| `]f`        | Go to next file alphabetically in current file's directory. In quickfix, equivalent to `:cnewer`     |
| `[n`        | Go to previous SCM conflict marker or diff/patch hunk                                                |
| `]n`        | Go to previous SCM conflict marker or diff/patch hunk                                                |

## Line operations

| Key binding | Description                                    |
| ----------- | ---------------------------------------------- |
| `[<space>`  | Add [count] blank line above                   |
| `]<space>`  | Add [count] blank line below                   |
| `[e`        | Exchange current line with [count] lines above |
| `]e`        | Exchange current line with [count] lines below |

## Option toggling

### Prefix

| Prefix | Description     |
| ------ | --------------- |
| `[o`   | Turn on option  |
| `]o`   | Turn off option |
| `yo`   | Toggle option   |

### Options mapping

| Mapping | Option                          |
| ------- | ------------------------------- |
| `b`     | `background`                    |
| `c`     | `cursorline`                    |
| `d`     | `diff` (`:diffthis`/`:diffoff`) |
| `e`     | `spell`                         |
| `h`     | `hlsearch`                      |
| `i`     | `ignorecase`                    |
| `l`     | `list`                          |
| `n`     | `number`                        |
| `r`     | `relativenumber`                |
| `u`     | `cursorcolumn`                  |
| `v`     | `virtualedit`                   |
| `w`     | `wrap`                          |
| `x`     | `cursorline` + `cursorcolumn`   |

## Paste operations

| Key binding | Description                                                    |
| ----------- | -------------------------------------------------------------- |
| `>p`        | Paste after line, increasing indent                            |
| `>P`        | Paste before line, increasing indent                           |
| `<p`        | Paste after line, decreasing indent                            |
| `<P`        | Paste before line, decreasing indent                           |
| `=p`        | Paste after line, reindenting                                  |
| `=P`        | Paste before line, reindenting                                 |
| `[op`       | Invoke `O` with paste on, leaving insert mode turns off paste  |
| `]op`       | Invoke `o` with paste on, leaving insert mode turns off paste  |
| `yop`       | Invoke `0C` with paste on, leaving insert mode turns off paste |

## Encode/decode

| Key binding                     | Description     |
| ------------------------------- | --------------- |
| `[x{motion}`/`[xx`/`{Visual}[x` | XML encode      |
| `]x{motion}`/`]xx`/`{Visual}]x` | XML decode      |
| `[u{motion}`/`[uu`/`{Visual}[u` | URL encode      |
| `]u{motion}`/`]uu`/`{Visual}]u` | URL decode      |
| `[y{motion}`/`[yy`/`{Visual}[y` | C string encode |
| `]y{motion}`/`]yy`/`{Visual}]y` | C string decode |
