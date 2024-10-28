<!-- markdownlint-disable MD013 -->

# Vim Macros

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Vim Macros](#vim-macros)
  - [Repeat](#repeat)
  - [Macro](#macro)
    - [Appending a recording](#appending-a-recording)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Repeat

| keystroke | action                                              |
| --------- | --------------------------------------------------- |
| `.`       | repeat last change                                  |
| `@:`      | repeat last command executed                        |
| `@@`      | repeat last `@` command executed (useful for macro) |

## Macro

1. `q<lowercase_letter>` begins recording keystrokes in a register identified by the lowercase letter
2. all keystrokes onward are recorded
3. `q` stops recording
4. `@<lowercase_letter>` execute keystrokes you've recorded

### Appending a recording

1. Hit `qa` to record keystrokes to register `a`, stop recording by hitting `q`.
2. Realize you forgot some keystrokes
3. Execute keystrokes to be sure you're at the last position of your recording.
4. Hit `qA` to append to register a. Hit `q` again to stop recording.
