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
with `split-window` or `new-window`:

~~~~
:respawn-pane
~~~~

A different command may be given as arguments:

~~~~
:respawn-pane top
~~~~

If a program is still running in the pane or window, the commands will refuse
to work. The `-k` flag kills the program in the window before starting the new
one:

~~~~
:respawn-pane -k top
~~~~

Like `split-window`, `respawn-pane` and `respawn-window` have a `-c` flag to
set the working directory.

<img src="images/tmux_remain_on_exit.png" align="right" width=368 height=235>

`respawn-pane` and `respawn-window` are useful with the `remain-on-exit`
option. When this is on, panes are not automatically killed when the program
running in them exits. Instead, a message is shown and the pane remains as it
was. This is called a dead pane, and `respawn-pane` or `respawn-window` can be
used to start the same or a different program.

#### Window sizes

Every window has a size, its horizontal and vertical dimensions. A window's
size is determined from the size of the clients attached to sessions it is
linked to. How this is done is controlled by the `window-size` option which may
be:

Value|Meaning
---|---
`largest`|The window has the size of the largest attached client; only part of the window is shown on smaller clients
`smallest`|The window has the size of the smallest attached client; on larger clients any unused space is filled with the `·` character
`latest`|The window has the size of the client which has been most recently used, for example by typing into it
`manual`|The window size is fixed; new windows use the `default-size` option and may be resized with the `resize-window` command

A window's size is not changed when it not linked to sessions that are
attached.

If a window has never been linked to an attached session - for example when
created as part of `new-session` with `-d` - it gets its size from the
`default-size` option. This is a session option with a default of 80x24:

~~~~
$ tmux show -g default-size
80x24
~~~~

When a session is created, its `default-size` option may be set at the same
time with the `-x` and `-y` flags:

~~~~~
$ tmux new -smysession -d -x160 -y48
$ tmux show -tmysession default-size
default-size 160x48
$ tmux lsw -tmysession
0: ksh* (1 panes) [160x48] [layout cc01,160x48,0,0,4] @4 (active)
~~~~~

When a window is larger than the client showing it, the visible area tracks the
cursor position. These keys may be used to view different areas of the window.

Key|Function
---|---
`C-b S-Up`|Move the visible area up
`C-b S-Down`|Move the visible area down
`C-b S-Left`|Move the visible area left
`C-b S-Right`|Move the visible area right
`C-b DC` (`C-b Delete`)|Return to tracking the cursor position

The visible area is a property of the client, so detaching the client or
changing the current window will reset to the cursor position. These keys are
bound to the `refresh-client` command.

A window size for an existing window may be set using the `resize-window`
commmand. This sets the size and automatically sets the `window-size` option to
`manual` for that window. For example:

~~~~
:resizew -x200 -y100
~~~~

To adjust the size up (`-U`), down (`-D`), left (`-L`) or right (`-R`):

~~~~
:resizew -L 20
~~~~

Or return to working out the size from attached clients:

~~~~
:resizew -A
~~~~

#### Session groups

XXX

#### Piping pane content

tmux allows any new changes to a pane to be piped to a command. This may be
used to, for example, make a log of a pane. The `pipe-pane` command does this:

~~~~
:pipe-pane 'cat >~/mypanelog'
~~~~

No arguments stops piping:

~~~~
:pipe-pane
~~~~

The `-I` flag to `pipe-pane` sends the output of a command to a pane. For
example this will send `foo` to the pane as if it had been typed:

~~~~
:pipe-pane -I 'echo foo'
~~~~

Used like this, `pipe-pane` with `-I` is similar to the `send-keys` command
covered in a later section.

The `-o` flag will toggle piping - starting if it is not already started,
otherwise stopping it. This is useful to start and stop from a single key
binding:

~~~~
bind P pipe-pane -o 'cat >~/mypanelog'
~~~~

#### Pane titles and the terminal title

Each pane in tmux has a title. A pane's title can be set by the program running
in the pane. If the program was running outside tmux it would set the outside
terminal title - normally shown in the *X(7)* window title. Because tmux can
have multiple programs running inside it, there is a pane title for each rather
than only one. The pane title is different from the window name which is used
only by tmux and is the same for all panes in a window.

Programs inside tmux can set the pane title using an escape sequence that looks
like this:

~~~~
$ printf '\033]2;title\007'
~~~~

tmux shows the pane title for the active pane in quotes on the right of the
status line.

The pane title for a pane can be changed from tmux using the `-T` flag to the
`select-pane` command:

~~~~
:selectp -Tmytitle
~~~~

However there is nothing to stop the program inside tmux changing the title
again after this.

tmux can set the outside terminal title itself, this is controlled by the
`set-titles` option:

~~~~
set -g set-titles on
~~~~

The default title includes the names of the attached session and current window
as well as the pane title for the active pane and the indexes of any windows
with alerts. This can be changed with the `set-titles-string` option which can
contain formats.

#### Mouse key bindings

tmux handles most mouse behaviour by mapping mouse events to key bindings.
Mouse keys have special names which are the event, followed by the button
number if any, then the area where the mouse event took place. For example:

- `MouseDown1Pane` for mouse button 1 pressed down with the mouse over a pane;

- `DoubleClick2Status` for mouse button 2 double-clicked on the status line;

- `MouseDrag1Pane` and `MouseDragEnd1Pane` for mouse drag start and end on a
  pane.

- `WheelUpStatusLeft` for mouse wheel up on the left of the status line

Terminals only support three buttons and the mouse wheel.

The possible mouse events are:

Event|Description
---|---
WheelUp|Mouse wheel up
WheelDown|Mouse wheel down
MouseDown|Mouse button down
MouseUp|Mouse button up
MouseDrag|Mouse drag start
MouseDragEnd|Mouse drag end
DoubleClick|Double click
TripleClick|Triple click

The possible areas where a mouse event may take place are:

Area|Description
---|---
Pane|The contents of a pane
Border|A pane border
Status|The status line window list
StatusLeft|The left part of the status line
StatusRight|The right part of the status line
StatusDefault|Any other part of the status line

Commands bound to a mouse key binding can use `-t` with the mouse target (`=`
or `{mouse}`) to tell tmux they want to use the pane or window where the mouse
event took place. For example this binds a double-click on the status line
window list to zoom the active pane of a window:

~~~~
bind -Troot DoubleClick1Status resizep -Zt=
~~~~

When the program running in a pane can itself handle the mouse, `send-keys` can
be used with the `-M` flag to pass the mouse event through to that program. The
`mouse_any_flag` format is true if the program has turned the mouse on. For
example, this binding makes button 2 paste, unless used over a pane which is in
a mode or where the program has enabled the mouse for itself:

~~~~
bind -Troot MouseDown2Pane selectp -t= \; if -F "#{||:#{pane_in_mode},#{mouse_any_flag}}" "send -M" "paste -p"
~~~~

#### The environment

XXX

#### Command aliases

tmux allows custom commands by defining command aliases. Note this is different
from the short alias of each command (such as `lsw` for `list-windows`).
Command aliases are defined with the `command-alias` server option. This is an
array option where each entry has a number.

The default has a few settings for convenience and a few for backwards
compatibility:

~~~~
$ tmux show -s command-alias
command-alias[0] split-pane=split-window
command-alias[1] splitp=split-window
command-alias[2] "server-info=show-messages -JT"
command-alias[3] "info=show-messages -JT"
command-alias[4] "choose-window=choose-tree -w"
command-alias[5] "choose-session=choose-tree -s"
~~~~

Taking `command-alias[4]` as an example, this means that the `choose-window`
command is expanded to `choose-tree -w`.

A custom command alias is added by adding a new index to the array. Because the
defaults start at index 0, it is best to use higher numbers for additional
command aliases:

~~~~
:set -s command-alias[100] 'sv=splitw -v'
~~~~

This option makes `sv` the same as `splitw -v`:

~~~~
:sv
~~~~

Any subsequent flags or arguments given to the entered command are appended to
the replaced command. This is the same as `splitw -v -d`:

~~~~
:sv -d
~~~~

### Scripting tmux

#### Basics of scripting

XXX

#### Unique identifiers

XXX

#### Command sequences

XXX

#### Command targets

XXX

#### Sending keys

XXX

#### Capturing pane content

XXX

#### Waiting, signals and locks

XXX

### Advanced configuration

#### Copy mode key bindings

XXX

#### Conditionals and {}

XXX

#### Array options

XXX

#### User options

XXX

#### Custom key tables

XXX

#### Automatic rename

XXX

#### terminfo(5) and terminal-overrides

XXX

#### XXX set-clipboard

XXX

#### XXX exit-empty

XXX
