<!-- markdownlint-disable MD013 -->

# flash.nvim

A hop plugin.

## `f`/`F`/`t`/`T` motions

Above keys are used to search single character within a line. This plugin expands these keys
to search across visible buffer.

Once selections are highlighted, for `f`/`F`, repeat motion with `f` or `F` to move between selections.
Same logic applies to `t`. Another method is to use `;`/`,` for next/previous match.

## Jump

To jump, hit binded key (e.g. `s`), enter pattern, and select labels to jump to.

## Treesitter integration

To highlight treesitter block scope that contains cursor position, hit binded key (e.g. `S`), enter pattern and select labels to jump to.
Use `;`/`,` to increase/decrease scope selection.

## Treesitter search

Treesitter integration requires cursor to be within the desired highlighted block scope.
With treesitter search, such restriction is no longer an issue.

This is usually used in operator-pending or visual mode. Hit key bind (e.g. `R`), enter search pattern,
select labels to jump to, then select the treesitter block scope to include.

## Remote search

Only useful in operator-pending mode, which allows selecting remote texts.
Hit key bind (`r`), enter pattern, select a label to select position, then enter a motion (flash jumps work too!)
Cursor should return to original position after remote search is complete.

## Search integration

During search, hit toggle search integration key binding (`<C-/>`), and select labels to jump to desired
search result.
