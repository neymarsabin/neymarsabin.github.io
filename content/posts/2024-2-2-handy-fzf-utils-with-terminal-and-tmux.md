+++
title = "fzf With terminal And tmux"
authors = ["neymarsabin"]
description = "fzf utils that I use daily..."
date = 2024-02-02T00:00:00+05:45
draft = false
+++

-   Install `tmux` if you haven't already! And the cheatsheets: [https://tmuxcheatsheet.com/](https://tmuxcheatsheet.com/)
-   [fzf](https://github.com/junegunn/fzf) improves your productivity! I wrote these scripts to add some improvements over my workflow. I had to fuzzy find tmux sessions, because navigating to my opened sessions was becoming tough. The idea is to find sessions like how my brain does.


## fuzzy find project and auto attach-session with tmux {#fuzzy-find-project-and-auto-attach-session-with-tmux}

```shell
#!/usr/bin/env sh
# Use at your own risk: I have seen bugs that create infinite tmux sessions :D
# define the fzf command: ./.fzfdir.sh pet
# syntax: ./.fzfdir.sh <working_directory>
# note: I have base_dir as ~/projects because I have all of my projects in ~/projects
# FZF_COMMAND="fzf-tmux"
FZF_COMMAND="fzf-tmux -p --with-nth 1"

# find in directories
workdir=$1
base_dir=~/projects
find_dir=$base_dir/$1

# execute command
RESULT=$(ls $find_dir | $FZF_COMMAND)

# create a new tmux session and attach to it
window_name=$RESULT
session_name="neymarsabin/$window_name"
workdir=$find_dir/$RESULT
send_command="cd $workdir"

if ! tmux has-session -t $session_name 2>/dev/null; then
  ## create new session, provide SESSION and WINDOW name
  new_session=$(TMUX= tmux new-session -A -d -s $session_name -n $window_name)

  ## switch and cd into project
  tmux switch-client -t $session_name
  tmux send-keys -t $session_name:$window_name "$send_command; clear" C-m
else
  tmux switch-client -t $session_name
fi
```


## fuzzy find sessions from within of $TMUX= sessions {#fuzzy-find-sessions-from-within-of-tmux-sessions}

```shell
#!/bin/bash
# ~/.fzftmux.sh
LIST_DATA="#{window_name} #{pane_title} #{pane_current_path}"
FZF_COMMAND="fzf-tmux -p --delimiter=: --with-nth 4 --color=hl:2"

TARGET_SPEC="#{session_name}:#{window_id}:#{pane_id}:"

# select pane
LINE=$(tmux list-windows -a -F "$TARGET_SPEC $LIST_DATA" | $FZF_COMMAND) || exit 0

# split the result
args=(${LINE//:/ })

# activate session/window/pane
tmux select-pane -t ${args[2]} && tmux select-window -t ${args[1]} && tmux switch-client -t ${args[0]}
```

And I did update my tmux config to set these as a keybindings.

```cfg
bind-key "s" run "~/.fzftmux.sh"
bind-key "p" run "~/.fzfdir.sh pet"
bind-key "y" run "~/.fzfdir.sh work"
bind-key "t" run "~/.fzfdir.sh oss"
```
