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

XXX

#### Working directories

XXX

#### Linking windows

XXX

#### Respawing panes and windows

XXX

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

#### XXX remain-on-exit

XXX

#### XXX exit-empty

XXX
