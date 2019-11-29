### Working with formats

Formats are a powerful way to get information about a running tmux server and
control the output of various commands.

They are used widely, for example:

- Displaying information (display-message).

- The format of text on the status lines, the terminal title
  (set-titles-string), and automatic rename (automatic-rename-format).

- The output of list commands (list-clients, list-commands, list-sessions,
  list-windows, list-panes, list-buffers).

- New window and session names (new-session, break-pane, new-window,
  split-window).

- Displayed text and filters in choose modes (choose-client, choose-tree,
  choose-buffer).

- Setting option values (set-option).

- Running commands (if-shell, run-shell).

- Menu content (display-menu).

This document gives a description of their use with examples.

Note that some of these features are only available in tmux 3.1 and later.

### Basic use

A format is a string containing special directives contained in `#{}` which
tmux will expand. Each `#{}` can reference named variables with some
information about the server, session, client or similar. Not all variables are
always present - an unknown or missing variable is replaced with nothing.

The simplest use is to display some information, for example to get the tmux
server PID using display-message (the -p flags prints the result rather than
displaying it in the status line):

~~~~
$ tmux display -p '#{pid}'
98764
~~~~

Or to modify the list-windows output:

~~~~
$ tmux lsw -F '#{window_id} #{window_name}'
@0 irssi
@1 mutt
@3 emacs
@4 ksh
~~~~

display-message is a useful command for working with formats. The `-a` flag
lists the formats it knows about:

~~~~
$ tmux display -a|fgrep width=
client_width=143
pane_width=143
window_width=143
~~~~

`-v` prints verbose information on how the format is evaluated to help spot
mistakes:

~~~~
$ tmux display -vp '#{pid}'
# expanding format: #{pid}
# found #{}: pid
# format 'pid' found: 98764
# replaced 'pid' with '98764'
# result is: 98764
98764
~~~~

Because `#` is special in formats, it needs to be doubled (`##`) to include a
`#`:

~~~~
$ tmux display -p '##{pid}'
#{pid}
~~~~

Many formats have a single-character alias, such as `#S` for `#{session_name}`,
but these can't be used with modifiers and their use is not encouraged.

### Options and environment variables

As well as format variables, a `#{}` in a format can contain the name of a tmux
option, such as a user option:

~~~~
$ tmux set @foo 'hello'
$ tmux display -p 'hello #{@foo}'
hello hello
~~~~

Or the name of an environment variable in the global environment:

~~~~
$ tmux showenv -g USER
USER=nicholas
$ tmux display -p '#{USER}'
nicholas
~~~~

Most of the examples below use user options (such as `@v`) but any format
variable or option or environment variable may be used instead.

### Simple modifiers

The result of expanding a variable can be changed with modifiers.

There are a few different forms of modifier, but most consist of an operator
with zero or more arguments separated by a punctuation character (`|` or `/`
are most usual), followed by a colon and one or more additional arguments which
are the variables or formats the modifier is applied to.

The simplest modifiers take no arguments and a single variable to modify. The
`t` modifier displays a time variable as a human readable string:

~~~~
$ tmux lsw -F '#{t:window_activity}'
Fri Nov 29 13:52:35 2019
Fri Nov 29 13:37:53 2019
Fri Nov 29 12:06:46 2019
Thu Nov 28 15:51:20 2019
Thu Nov 28 07:41:05 2019
~~~~

`b` and `d` trim the file and directory name from a path:

~~~~
$ tmux set @p `pwd`
$ tmux display -p '#{d:@p}'
/usr/src/usr.bin
$ tmux display -p '#{b:@p}'
tmux
~~~~

### Trimming and padding

Format variables may be trimmed or padded to size. The `=` modifier trims and
the `p` modifier pads. They both take at least one argument which is the width
- a positive width means to trim on the left or pad on the right and a negative
the opposite:

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=3:@v}'
foo
$ tmux display -p '#{=-3:@v}'
bar
$ tmux display -p '#{p9:@v}baz'
foobar   baz
$ tmux display -p '#{p-9:@v}baz'
   foobarbaz
~~~~

Multiple modifiers can be applied together by separating them with a `;`, for
example:

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=3;p-6:@v}'
   foo
~~~~

The `=` trim modifier accepts a second argument which is a string to append or
prepend to the result to show it has been trimmed. When giving more than one
argument to a modifier, they must be separated by a punctuation character,
including one right after the operator. `/` or `|` are most common separators:

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=|6|...:@v}'  # nothing trimmed
foobar
$ tmux display -p '#{=|5|...:@v}'  # trimmed on the right
fooba...
$ tmux display -p '#{=|-5|...:@v}' # trimmed on the left
...oobar
~~~~

### Comparisons

For comparison purposes, tmux considers the result of a format to be either
true or false. The result is true if it is not empty and not a single zero
(`0`), otherwise it is false.

There are several comparison operators available. Rather than the single
variable given to `t` and `b`, these take two variables to compare, separated
by a comma. Both of these may be formats themselves rather than variables.

`==`, `!=`, `<`, `>`, `<=` and `>=` are string comparisons:

~~~~
$ tmux set @v foo
$ tmux display -p '#{==:#{@v}bar,foobar}'
1
$ tmux display -p '#{!=:#{@v}bar,foobar}'
0
$ tmux display -p '#{<:#{@v},bar}'
0
~~~~

`||` is true if either of its arguments are true and `&&` if both are:

~~~~
$ tmux set @v foo
$ tmux display -p '#{||:0,#{@v}}'
1
$ tmux display -p '#{&&:0,#{@v}}'
0
~~~~

A ternary choice operator `?` is also available. This is slightly different
from the other comparison modifiers (it was implemented earlier) and has no
colon between the operator and the condition to check. The condition is
followed by two result formats, the first is chosen if the condition is true
and the second if it is false. Either may be empty. For example:

~~~~
$ tmux set @v 0
$ tmux display -p '#{?@v,yes,no}'
no
$ tmux display -p '#{?#{==:#{@v},0},yes,no}'
yes
$ tmux display -p '#{?#{==:#{@v},0},#{@v} is true,#{@v} is false}'
0 is true
~~~~

Inside the results of the choice operator, a comma can be inserted by escaping
it as `#,`.

### Substitution

Formats support substitution through the `s` modifier. This is similar to
sed(1) substitution and takes two or three arguments - a regular expression to
search for, the string to replace it with and a set of flags. Both the regular
expression to search for and the string to replace with may be formats
themselves. Patterns in brackets are expanded in the replacement by number
(`\1`, `\2` and so on).

Like `t` and `b` and `d`, the variable being replaced cannot be a format but
must be a variable:

~~~~
$ tmux set @v foobar
$ tmux display -p '#{s|foo|bar|:@v}'
barbar
$ tmux display -p '#{s|(foo)(bar)|\2\1|:@v}'
barfoo
$ tmux set @w foo
$ tmux display -p '#{s|#{@w}|xxx|:@v}'
xxxbar
~~~~

The third argument supports one flag: `i` which means the regular expression
should be case insensitive:

~~~~
$ tmux set @v foobar
$ tmux display -p '#{s|FOO|xxx|:@v}'
foobar
$ tmux display -p '#{s|FOO|xxx|i:@v}'
xxxbar
~~~~

### Matching and searching

The matching modifier `m` is similar to the comparison modifiers and compares
two formats - the first is a pattern matched against the second. By default
this expects an fnmatch(3) pattern, but the `r` flag specifies a regular
expression. The `i` flag means case insensitive:

~~~~
$ tmux set @v foobar
$ tmux display -p '#{m:*foo*,#{@v}}'
1
$ tmux display -p '#{m/ri:^FOO,#{@v}}'
1
~~~~

The `C` modifier searches for the given format in the pane content and returns
the line number or zero if it is not found. 

~~~~
$ # xyz
$ tmux display -p '#{C:x*z}'
4
$ tmux display -p '#{C/r:x.z}'
2
~~~~

### Loops

The `S`, `W` and `P` modifiers look over every session, every window in the
current session and every pane in the current window and expand the given
format for each. `W` or `P` may be given a second format which is used for the
current window and active pane. For example:

~~~~
$ tmux display -p '#{W:#{window_name} }'
irssi mutt emacs ksh ksh ksh ksh ksh ksh emacs emacs ksh ksh ksh ksh ksh
$ tmux display -p '#{W:#{window_name} ,--- }'
irssi mutt emacs ksh ksh ksh ksh ksh --- emacs emacs ksh ksh ksh ksh ksh
~~~~

### Literals and quoting

The `l` modifier results in the literal string given to it, this can be used
for a literal as the first argument to the `?` choice modifier, for example:

~~~~
$ tmux display -p '#{?#{l:1},a,b}'
a
~~~~

The `q` modifier quotes any special characters:

~~~~
$ tmux set @v '()'
$ tmux display -p '#{q:@v}'
\(\)
~~~~

### Multiple expansion

tmux has two modifiers which expand their result twice: `E` and `T`.
`#{E:status-left}` will expand the contents of the `status-left` option. `T`
also expands `strftime(3)` conversion specifiers (like `%H` and `%M`).

~~~~
$ tmux display -p '#{status-left}'
[#{session_name}]
$ tmux display -p '#{T:status-left}'
[0]
~~~~

### Tree mode and formats

XXX

### Formats as filters

XXX

### Summary of modifiers

The table below shows the syntax of the modifiers. The following fields are
used:

- `variable` means a variable name only, such as `session_name` or `@foo`. This
  is not expanded and can't contain `#{}`.

- `format` means a string fully expanded as a format, such as `abc` or
  `abc#{@foo}`. Variables must be included in `#{}` or they are not expanded:
  `session_name` is the literal string `session_name`, it must be
  `#{session_name}` to expand it.

- `single` means a single format or a variable, but not a string, so `#{session_name}`
  or `session_name` will both be expanded but `abc#{session_name}` will not.

- `string` means a string that is not expanded, such as `abc`.

- `flags` is an optional set of single character flags.

Format|Description
---|---
`#{t:variable}`|Time to human readable string
`#{b:variable}`|File name of path
`#{d:variable}`|Directory name of path
`#{=N:variable}`|Trim to width N
`#{=/N/format:variable}|Trim to width N with a marker if trimmed
`#{pN:variable}`|Pad to width N
`#{==:format,format}`|Compare two formats (also `!=` `<` `>` `<=` `>=`)
`#{?single,format,format}`|Choose from two formats
`#{m/flags:format,format}`|Match a pattern against a format, flags `r` and `i`
`#{C/flags:format}`|Search for a format, flags `r` and `i`
`#{P:format,format}`|Loop over each pane (also `S`, `W`)
`#{l:string}`|Literal string
`#{E:variable}`|Expand variable content
`#{T:variable}`|Expand variable content with time conversion specifiers
`#{q:variable}`|Quote special characters
