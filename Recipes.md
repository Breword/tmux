## Configuration file recipes

This file lists some useful configuration file snippets to control tmux
behaviour.

### Prevent pane movement wrapping

This stops the pane movement keys wrapping around at the top, bottom, left and
right.

Requires tmux 2.6 or later.

~~~~
bind -r Up if -F '#{pane_at_top}' '' 'selectp -U'
bind -r Down if -F '#{pane_at_bottom}' '' 'selectp -D'
bind -r Left if -F '#{pane_at_left}' '' 'selectp -L'
bind -r Right if -F '#{pane_at_right}' '' 'selectp -R'
~~~~

### Make `C-b w` binding only show the one session

This makes the `C-b w` tree mode binding only show windows in the attached
session.

~~~~
bind w run 'tmux choose-tree -Nwf"##{==:##{session_name},#{session_name}}"'
~~~~

### Create a new pane to copy

This opens a new pane with the history of the active pane - useful to copy
multiple items from the history to the shell prompt.

Requires tmux 3.2 or later.

~~~~
bind C {
	splitw -f -l30% ''
	set-hook -p pane-mode-changed 'if -F "#{!=:#{pane_mode},copy-mode}" "kill-pane"'
	copy-mode -s'{last}'
}
~~~~
