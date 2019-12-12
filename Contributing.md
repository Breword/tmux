## Source code and development process

tmux is part of the OpenBSD base system and OpenBSD CVS is the primary source
repository:

https://cvsweb.openbsd.org/cgi-bin/cvsweb/src/usr.bin/tmux/

GitHub holds the portable tmux version. There are a few minor differences (for
example, the OpenBSD code doesn't include the -V flag) but otherwise the
additions are mainly for portability.

Code changes to the main tmux code are committed to OpenBSD and then a script
automatically applies any new commits to the GitHub repository every few hours.

It is fine to develop and submit code changes with a GitHub PR, but the final
form will be a patch file which is applied to OpenBSD CVS.

## Releases

tmux currently sees a new release approximately every six months - the same
schedule as OpenBSD, around May and October. This means that the GitHub
releases contain roughly the same code as OpenBSD releases (but not necessarily
exactly the same).

The release process consists of a branch from which one or more release
candidates are produced for testing, until the final release is published and
the branch removed.

## Non-code contributions

Here are some ideas for anyone who would like to contribute to tmux without
writing any code:

- Testing. Test master, test release candidates, test releases.

- Improve the FAQ. Write (and maintain) some official howto guides.

- Help with packaging releases for different platforms or produce static builds
  so people can test more easily.

- CI for more platforms.

- Write regression tests (see the regress/ directory).

- Write (and maintain) build and installation instructions for different platforms.

- Triage and manage open issues and help to reply.

## Code contributions

Here is a list of outstanding feature requests or notes for future
development. They are sorted into three sections approximately by difficulty of
implementation. If there is a GitHub issue then its number is shown in
brackets.

It is worth getting in touch before starting work, particularly on larger
items, to avoid any duplication of effort.

### Small things

- "After" hooks are missing for many commands that do not use CMD_AFTERHOOK.

- Some sort of menu or menus in copy mode.

- A command in copy mode to toggle the selection.

- wait-for could do more, for example being able to wait for a pane to exit or
  close (could use the existing notify code in some way). Also a flag for a
  timeout or to stop waiting on a signal.

- ([#1784](https://github.com/tmux/tmux/issues/1784)) A way to disable line
  wrap but preserve any trimmed content (so it can be viewed if the pane is
  made bigger).

- Timeout to command-prompt, if the timeout expires the command is run with
  whatever is in the prompt at the time. This would allow numbers to be bound
  for example "bind 2 command-prompt -I2 -T50 select-window -t':%%'" and the
  user has 50 milliseconds to enter window 23 or it will go to window 2.

### Medium things

- Moving, joining and otherwise reorganizing panes, windows and session should
  be easier in tree mode. For example, either a new key to swap tagged panes if
  two are tagged, or tagged pane and current if one is tagged, and so on. Or
  make :swap-pane use tagged panes but that might be much harder. Likewise for
  move, join, etc.

- Regex search and highlighting in copy mode. Could also look at cleaning up
  and perhaps merging the various stringify and search bits.

- ([#1605](https://github.com/tmux/tmux/issues/1605)) Support for ZERO WIDTH
  JOINER U+200D.

- Marked positions in history. Could use the same prompt-detection escape
  sequences as iTerm2. Could be listed by capture-pane and also a menu to jump
  to marks in copy mode. ([#1042](https://github.com/tmux/tmux/issues/1042)) is
  related and also has some code to display a marker line.

- ([#1545](https://github.com/tmux/tmux/issues/1545)) Copy mode searching is
  very slow when there is a big history, need a good solution.

- Drag panes and windows around in tree mode in order to move or swap them.

- Make the commmand prompt able to take up multiple lines.

- ([#918](https://github.com/tmux/tmux/issues/918)) A way to specify how panes
  are merged when one is killed. Could be an option to kill-pane.

- Allow multiple targets either with multiple -t or by giving a pattern or both.

- Searching in copy mode should handle wrapping, so a search for "foobar" then
  it should be found even if wrapped into "foo\nbar" (that is, the
  GRID_LINE_WRAPPED flag is set on the line).

- ([#682](https://github.com/tmux/tmux/issues/682)) Improve word and line
  selection in copy mode (for example when dragging it should select by
  word. Compare how xterm(1) works.

- ([#1718](https://github.com/tmux/tmux/issues/1718)) Copy mode should behave
  better if the pane outside is modified. At the moment it stops reading which
  is not ideal, and there are also questions about how it should work with
  copy-mode -e.

- The lexer in cmd-parse.y should be a single state machine rather than separate
  functions for environment variables, strings and formats.

- ([#1842](https://github.com/tmux/tmux/issues/1842)) Floating windows
  (wouldn't call them windows though). Could use same overlay mechanism as
  menus, would need a way to update the content, could just be a separate
  process with its own screen or could be a command.

- ([#1868](https://github.com/tmux/tmux/issues/1868)) Vertical-only zoom.

- ([#1774](https://github.com/tmux/tmux/issues/1774)) Resizing panes should
  move to the parent cell and resize it if this would allow the pane to
  become closer to what is requested.

### Large things

- Better layouts. For example it would be good if they were driven by hints
  rather than fixed positions and could be automatically reapplied after
  resize/split/kill. Pane options can be used.

- ([#1269](https://github.com/tmux/tmux/issues/1269)) Store grids in
  blocks. Can be used to reflow on demand. Would be nice to revisit how
  history-limit works - would it be better as a global limit rather than per
  pane?

- ([#1503](https://github.com/tmux/tmux/issues/1503)) Panes that cross multiple
  columns for extra height. This seems fraught with complexity for anything but
  the simplest cases.

- Link panes into multiple windows.

- Separate active panes for different clients.

- ([#44](https://github.com/tmux/tmux/issues/44)) &
  ([#1613](https://github.com/tmux/tmux/issues/1613)) Support for SIXEL.
