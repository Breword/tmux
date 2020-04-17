## The clipboard

It is common to want to have text copied from tmux's copy mode or with the
mouse in tmux synchronized with the system clipboard. The tools offered to tmux
by terminals to do this are quite blunt and not consistently supported. This
document gives an overview of how things work and some configuration examples.

### The `set-clipboard` option

#### How it works

Some terminals offer an escape sequence to set the clipboard. This is one of
the operating system control sequences so it is known as OSC 52

The way it works is that when text is copied in tmux it is packaged up and sent
to the outside terminal in a similar way to how tmux draws the text and colours
and attributes. The outside terminal recognises the clipboard escape sequence
and sets the system clipboard.

The big advantage of `set-clipboard` is that it works over an *ssh(1)*
connection even if X11 forwarding is not configured. The disadvantages are that
it is patchily supported and can be tricky to configure.

tmux supports this through the `set-clipboard` option. For it to work, three
things must be in place:

1. The `set-clipboard` option must be set to `on` or `external`. The default is
   `external`.

2. The `Ms` capability must be available to tmux when it looks at the
   *terminfo(5)* entry specified by `TERM`. This is present by default for some
   terminals and if not is added with `terminal-overrides` (shown below).

3. The feature must be enabled in the terminal itself. How this is done varies
   from terminal to terminal. Some have it enabled by default and some do not.

The following two sections show how to configure `set-clipboard` and `Ms`;
later sections cover support in different terminals.

#### Changing `set-clipboard`

The tmux `set-clipboard` option was added in tmux 1.5 with a default of `on`;
the default was changed to `external` when `external` was added in tmux 2.6.

The difference is that `on` both makes tmux set the clipboard for the outside
terminal, and allows applications inside tmux to set tmux's clipboard (adding a
paste buffer). `external` only makes tmux set the clipboard and forbids
applications inside from doing so.

Any of these commands in `.tmux.conf` or at the command prompt will set the
three states:

~~~~
set -g set-clipboard on
set -g set-clipboard external
set -g set-clipboard off
~~~~

#### Setting the `Ms` capability

By default, tmux adds the `Ms` capability for terminals where `TERM` matches
`xterm*`. `TERM` can be checked with this command run outside tmux:

~~~~
echo $TERM
~~~~

To see if `Ms` is already set, run this from *inside* tmux. If `Ms` is shown
like this, it does not need to be set with `terminal-overrides`. If it shows
`[missing]`, then it must be configured with `terminal-overrides`.

~~~~
$ tmux info|grep Ms:
 180: Ms: (string) \033]52;%p1%s;%p2%s\a
~~~~

If `Ms` is missing, it can be added with `terminal-overrides`. First check what
`TERM` is outside tmux:

~~~~
$ echo TERM
rxvt-unicode-256color
~~~~

Then add an appropriate `terminal-overrides` line to `.tmux.conf`. Change
`rxvt-unicode-256color` to the appropriate `TERM`:

~~~~
set -as terminal-overrides ',rxvt-unicode-256color:Ms=\E]52;%p1%s;%p2%s\007'
~~~~

Multiple similar lines may be added to `.tmux.conf` for different values of
`TERM` (`-a` means to append to the option). It also supports wildcards, so
using `rxvt-unicode*` would apply to both `rxvt-unicode` and
`rxvt-unicode-256color`.

#### Security concerns

If `set-clipboard` is set to `external`, only tmux can set the clipboard. If
set to `on` and tmux is version 2.6 or newer, any application running inside
tmux can set the system clipboard.

#### Terminal support - tmux inside tmux

XXX

#### Terminal support - xterm

xterm supports OSC 52 but it is disabled by default. It can be enabled by
putting this in `.Xresources` or `.Xdefaults`:

~~~~
XTerm*disallowedWindowOps: 20,21,SetXprop
~~~~

#### Terminal support - VTE terminals

VTE terminals (GNOME terminal, XFCE terminal, Terminator) do not support the
OSC 52 escape sequence.

Most will ignore it, but some versions will not and will instead print it to
the terminal - this appears as a large set of letters and numbers covering any
existing text. To fix this problem, turn `set-clipboard` off:

~~~~
set -g set-clipboard off
~~~~

#### Terminal support - kitty

Kitty does support OSC 52, but it has a bug where it appends to the clipboard
each time text is copied rather than replacing it.

#### Terminal support - rxvt-unicode

rxvt-unicode does not support OSC52 natively. There is an unofficial Perl
extension [here](http://anti.teamidiot.de/static/nei/*/Code/urxvt/).

#### Terminal support - st

st supports OSC 52 but has a length limit on the amount of text that can be
copied, so text may be truncated.

#### Terminal support - iTerm2

iTerm2 supports OSC 52 but it has be enabled from the preferences with this
option:

<img src="images/iterm2_clipboard.png" align="center" width=378 height=201>

#### Quick summary

In summary, to configure `set-clipboard`, follow these steps:

1. Make sure `set-clipboard` is set in tmux:

    ~~~~
    $ tmux show -g set-clipboard
    external
    ~~~~

    If it is not `on` or `external`, add this to `.tmux.conf` and restart tmux
    (use `on` rather than `external` before tmux 2.6):

    ~~~~
    set -g set-clipboard external
    ~~~~

2. Make sure `Ms` is set. Start tmux and run:

   ~~~~
   $ tmux info|grep Ms
   180: Ms: [missing]
   ~~~~

   If it is `[missing]`, get the value of `TERM` outside tmux:

   ~~~~
   $ echo $TERM
   rxvt-unicode-256color
   ~~~~

   Then add an appropriate `terminal-overrides` line to `.tmux.conf` and
   restart tmux:
   
   ~~~~
   set -as terminal-overrides ',rxvt-unicode-256color:Ms=\E]52;%p1%s;%p2%s\007'
   ~~~~

   Then start tmux and check it has worked by running this inside tmux:

   ~~~~
   $ tmux info|grep Ms:
   180: Ms: (string) \033]52;%p1%s;%p2%s\a
   ~~~~

3. Enable support in the terminal options if necessary, or use a terminal where
   it is enabled by default.

### External tools

XXX

#### Common issues - `DISPLAY`

XXX

#### Common issues - X11 forwarding

XXX

#### Common issues - xclip

XXX
