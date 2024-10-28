<!-- markdownlint-disable MD013 -->

# nvim-surround

## Add surround (normal mode)

| key binding        | descriptions                                                                 |
| ------------------ | ---------------------------------------------------------------------------- |
| `ys{motion}{char}` | surround text in a given `motion` with delimiter pair associated with `char` |
| `yss{char}`        | surround current line with delimiter pair associated with `char`             |
| `yS{motion}{char}` | analogous to `ys`, but add delimiter on new lines                            |
| `ySS{char}`        | analogous to `yss`, but add delimiter on new lines                           |

## Delete surround (normal mode)

| key binding | descriptions                                          |
| ----------- | ----------------------------------------------------- |
| `ds{char}`  | delete surround delimiter pair associated with `char` |
| `dss`       | delete closest surround delimiter pair detected       |

## Change surround (normal mode)

| key binding               | descriptions                                                                                                      |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `cs{target}{replacement}` | change surround delimiter pair associated with `target` char to delimiter pair associated with `replacement` char |
| `css{char}`               | change closest surround delimiter pair detected to delimiter pair associated with `char`                          |
| `cS{target}{replacement}` | analogous to `cs`, but add delimieter pair on new lines                                                           |

## Add surround (visual mode)

| key binding | descriptions                                                                      |
| ----------- | --------------------------------------------------------------------------------- |
| `S{char}`   | in visual mode, surround selected text with delimiter pair associated with `char` |

- charwise-visual mode: add pair around selection
- linewise-visual mode: add pair around selection on new lines
- blockwise-visual mode: add pair around visual block, once per line

## Brackets

For bracket-like pairs with open/close pairs, e.g. `()`, `[]`, `{}`, `<>`,
closing character adds surround with just the pair, while opening character
adds a whitespace between selection and the pair.

## HTML tags

HTML tags are triggered with `t` or `T`. `t` and `T` are identical, except
`cst` will only change surrounding tag's type, while `csT` will change the
entire tag's content.

## Functions

Functions are triggered with `f`

Examples:

| old text        | command           | new text            |
| --------------- | ----------------- | ------------------- |
| `args`          | `ysiwffunc`       | `func(args)`        |
| `func(a, b, c)` | `dsf`             | `a, b, c`           |
| `func(a, b, c)` | `csfnew_func<cr>` | `new_func(a, b, c)` |

## Insert key

Insert key are triggered with `i`, which queries user for left/right hand side pair.

| old text | command          | new text |
| -------- | ---------------- | -------- |
| `text`   | `yssi/<cr>\<cr>` | `/text\` |

## Jumps

If cursor is not inside a surrounding pair, it can jump to "nearest" pair. Nearest pair precedence is:

1. pairs that surround the cursor
2. pairs after the cursor
3. pairs before the cursor
