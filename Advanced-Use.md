## Advanced use

### About this document

This document gives a description of some of tmux's more advanced features and
some examples. It is split into three sections covering:

* features most useful when using tmux interactively;

* those for scripting with tmux;

* and advanced configuration.

However, many of the features discussed are useful both interactively and when
scripting.

### Using tmux

#### Socket and multiple servers

tmux creates a directory for the user in `/tmp` the server then creates a
socket in that directory. The default socket is called `default`, for example:

~~~~
$ ls -l /tmp/tmux-1000/default
srw-rw----  1 nicholas  wheel     0B Mar  9 09:05 /tmp/tmux-1000/default=
~~~~

Sometimes it is convenient to create separate tmux servers, perhaps to ensure
an important process is completely isolated or to test a tmux configuration.
This can be done by using the `-L` flag which creates a socket in `/tmp` but
with a name other than `default`. To start a server with the name `test`:

~~~~
$ tmux -Ltest new
~~~~

Alternatively, tmux can be told to use a different socket file outside `/tmp`
with the `-S` flag:

~~~~
$ tmux -S/my/socket/file new
~~~~

The socket used by a running server can be seen with the `socket_path` format.
This can be printed using the `display-message` command with the `-p` flag:

~~~~
$ tmux display -p '#{socket_path}'
/tmp/tmux-1000/default
~~~~

If the socket is accidentally deleted, it can be recreated by sending the
`USR1` signal to the tmux server:

~~~~
$ pkill -USR1 tmux
~~~~

#### Alerts and monitoring

An alert is a way of notifying the user when something happens in a pane in a
window. tmux supports three kinds of alerts:

* Bell: when the program sends an ASCII `BEL` character. This is turned on or
  off with the `monitor-bell` option.

* Activity: when any output is received from the program. This is turned on or
  off with the `monitor-activity` option.

* Silence: when no output is received from the program. A time period in
  seconds during which there must be no output is set with the
  `monitor-silence` option. A period of zero disables this alert.

<img src="images/tmux_alert_flags.png" align="right" width=368 height=235>

An alert in a pane does two things for each session containing the pane's
window.

Firstly, it sets a flag on the window in the window list, but only if the window is not
the current window. While this flag is set:

* The window is drawn in the window list using the style in the
  `window-status-bell-style` (for bell) or `window-status-activity-style` (for
  activity and silence) options. The default is to use the reverse attribute.


* The window name is followed by a `!` for bell, a `#` for activity and a `~`
  for silence.

Alert flags on a window are cleared as soon as the window becomes the current
window. All flags in a session may be cleared by using `kill-session` with the
`-C` flag:

~~~~
:kill-session -C
~~~~

<img src="images/tmux_alert_message.png" align="right" width=368 height=235>

The `C-b M-n` and `C-b M-p` key bindings move to the next or previous window
with an alert, using the `-a` flag to the `next-window` and `previous-window`
commands.

Secondly, it may show a message in the status line, sound a bell in the outside
terminal, or both. Whether this is a bell or a message is controlled by the
`visual-bell`, `visual-activity` and `visual-silence` options. The choice of
when to take this action is controlled by the `bell-action`, `activity-action`
and `silence-action` options which may be:

Value|Meaning
---|---
`any`|An alert in any window in the session triggers an action
`none`|No action is triggered in the session
`current`|An alert is triggered for a bell, activity or silence in the current window but not other windows
`other`|An alert is triggered for a bell, activity or silence in any window except the current window

#### Working directories

Each tmux session has default working directory. This is the working directory
used for each new pane.

A session's working directory is set when it is first created:

* It may be given with the `-c` flag to `new-session`, for example:

~~~~
$ tmux new -c/tmp
~~~~

* If the session is created from a key binding or from the command prompt, it
  is the working directory of the attached session

* If the session is created from the shell prompt inside or outside tmux, it is
  the working directory of the shell.

A session's working directory may be changed with the `-c` flag to
`attach-session`, for example:

~~~~
:attach -c/tmp
~~~~

When a window or pane is created, a working directory may be given with `-c` to
`new-window` or `split-window`. This is used instead of the session's default
working directory:

~~~~
:neww -c/tmp
~~~~

Or:

~~~~
:splitw -c/tmp
~~~~

tmux can try to read the current working directory of a pane from outside the
pane. This is available in the `pane_current_path` format. This changes the
`C-b "` binding to create a new pane with the same working directory as the
active pane:

~~~~
bind '"' splitw -c '#{pane_current_path}'
~~~~

#### Linking windows

XXX

#### Respawing panes and windows

Respawning a pane or window is a way to start a different (or restart the same)
program without need to recreate the window, maintaining its size, position and
index.

The `respawn-pane` command respawns a pane and `respawn-window` a window. By
default, they run the same program as the pane or window as initially created
with using `split-window` or `new-window`:

~~~~
:respawn-pane
~~~~

A different command may be given as arguments:

~~~~
:respawn-pane top
~~~~

If a program is still running in the pane or window, the commands will refuse
to work. The `-k` flag forces this and kills the program in the window before
starting the new one:

~~~~
:respawn-pane -k top
~~~~

<img src="images/tmux_remain_on_exit.png" align="right" width=368 height=235>

`respawn-pane` and `respawn-window` are useful when used with the
`remain-on-exit` option. When this is on, panes are not automatically killed
when the program running in them exits. Instead, a message is shown and the
pane remains as it was. This is called a dead pane, and `respawn-pane` or
`respawn-window` can be used to start the same or a different program again.

#### Window sizing

XXX

#### Session groups

XXX

#### Capturing pane content

XXX

#### Piping pane content

XXX

#### Pane titles

XXX

#### Key tables

XXX

#### Mouse key bindings

XXX

#### The environment

XXX

#### Command aliases

XXX

### Scripting tmux

#### Basics of scripting

XXX

#### Unique identifiers

XXX

#### Command sequences

XXX

#### Command targets

XXX

#### The `wait-for` command

XXX

### Advanced configuration

#### Conditionals and {}

XXX

#### Array options

XXX

#### User options

XXX

#### Automatic rename

XXX

#### terminfo(5) and terminal-overrides

XXX


#### XXX exit-empty

XXX
