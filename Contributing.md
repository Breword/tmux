## Source code and development process

tmux is part of the OpenBSD base system and OpenBSD CVS is the primary source
repository:

https://cvsweb.openbsd.org/cgi-bin/cvsweb/src/usr.bin/tmux/

GitHub holds the portable tmux version. There are a few minor differences,
mostly for portability.

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

## Code contributions

Here is a list of outstanding feature requests or notes for future development.
They are sorted into three sections approximately by difficulty of
implementation. If there is a GitHub issue then its number is shown in
brackets.

It is worth getting in touch before starting work, particularly on larger
items, to avoid any duplication of effort.

### Small things

- It is annoying that -t= is still needed for select-window on the status line
  when it is not needed for panes.

- "After" hooks are missing for many commands that do not use CMD_AFTERHOOK.

- A command in copy mode to toggle the selection.

- It would be nice to have some more preset layouts.

- In emacs cursor movement cancels incremental search, tmux should work the
  same way.

- ([#2205](https://github.com/tmux/tmux/issues/2205))In copy mode, should add
  incremental search with regex (new commands).

- Customize mode: 1) way to export option or tagged options to a file 2) `e`
  key like in buffer mode to edit option file in an editor popup 3) way to add
  new user options 4) way to add new key bindings 5) a key to restore an
  option, a key, or an entire key table to its default.

### Medium things

- In copy mode - should the bottom be the last used line? It can be annoying to
  have to move the cursor through a load of empty space. It might be better to
  draw a line at the bottom (already have code for this) and prevent the cursor
  moving below it.

- Completion at the command prompt could be more clever: it could recognise
  commands and have some way to describe their arguments so for example only
  complete options for set-option and layouts for select-layout; it could
  complete -t with a space after it as well as without; it could complete
  special targets like {left}; it could complete panes as well as windows.
  Probably lots more things.

- At the moment, the optimization in screen-write.c is purely on lines
  scrolled:
  
  - If there are multiple changes within a line they will both be written even
    if they are on top of each other. We could trim text strings that overlap.
  
  - ECH, DCH and friends cause a flush when they could just remove the
    characters from the write buffer?

  - We could check the existing screen and drop characters that are unchanged.

  But is it worth the cost? Would it make any difference to any applications?

- Support DECSLRM margins within tmux itself.

- Could the history be a ring buffer? Would it be better? It doesn't help us
  with a scroll region or in the alternate screen.

- list-keys should be able to show long commands with {} and newlines more
  nicely. Could store knowledge of command arguments in cmd_entry like a string
  with a character per argument - so if-shell might be "sTT" for shell-command
  and two tmux command. Would need syntax to express the different forms -
  new-window, bind-key, run-shell, if-shell, display-menu. Also what about
  stuff like popup -R and detach -E?

- Should remember the last layout before select-layout was used and
  select-layout without an argument should include it, so C-Space could cycle
  through it with the preset layouts. Also a separate flag or layout name to
  restore it directly. Or how about a layout history and a way to move back? A menu?

- Should last pane be a stack like windows?

- wait-for could do more, for example being able to wait for a pane to exit or
  close (could use the existing notify code in some way). Also a flag for a
  timeout or to stop waiting on a signal.

- ([#1784](https://github.com/tmux/tmux/issues/1784)) A way to disable line
  wrap but preserve any trimmed content (so it can be viewed if the pane is
  made bigger).

- Moving, joining and otherwise reorganizing panes, windows and session should
  be easier in tree mode. This should use the marked pane rather than mixing it
  up with tagging. Maybe keys to break/join/move without leaving tree mode?
  Also dragging would be nice.

- ([#1605](https://github.com/tmux/tmux/issues/1605)) Support for ZERO WIDTH
  JOINER U+200D.

- Marked positions in history. Could use the same prompt-detection escape
  sequences as iTerm2. Could be listed by capture-pane and also a menu to jump
  to marks in copy mode. ([#1042](https://github.com/tmux/tmux/issues/1042)) is
  related and also has some code to display a marker line.

- A way to set a mark in copy mode and commands to jump back to it - "set-mark"
  and "jump-to-mark" commands.

- Make the commmand prompt able to take up multiple lines.

- ([#918](https://github.com/tmux/tmux/issues/918)) A way to specify how panes
  are merged when one is killed. Could be an option to kill-pane.

- Allow multiple targets either with multiple -t or by giving a pattern or both.

- It would be nice to be able to remember the position in copy mode and go back
  to the same place when entering it again. How would this work if the pane
  scrolls? What about entering with the mouse?

- It would be nice to have commands to build a paste buffer in copy mode by
  doing multiple copies. It would need to display the work in progress
  somewhere (bottom left?) and have a command to add a chunk of text and a
  command to remove the last chunk and a command to clear.

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

- Link panes into multiple windows.

- Separate active panes for different clients.

- ([#44](https://github.com/tmux/tmux/issues/44)) &
  ([#1613](https://github.com/tmux/tmux/issues/1613)) Support for SIXEL.
