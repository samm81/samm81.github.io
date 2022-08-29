---
layout: post
title: `tmux link-window`
---

`tmux` is my [multiplexer][1] of choice, and I love it, but a part of the workflow didn't work quite right for me. I use [org-mode][2] for emacs and would keep a dedicated [session][3] open with just a single window in which I had `emacs` up. Context switching from other work to org (to clock in or out of a task, for example) would require `Prefix s` (I use `C-q` for my [prefix][4]) -> navigate to org session -> do org work -> `Prefix s` -> navigate back to work session.

https://user-images.githubusercontent.com/1221372/187302277-2ab7ae96-d35a-497a-9480-853c7c6e96dc.mp4

[My brother][5] was recently working on writing a custom layout for [xmonad][6] with a 2D grid of windows that featured "pinned" columns. So rows would function as different contexts or workspaces, and columns within rows would be windows within that workspace, but certain columns would be shared across all rows ("pinned"), so that he could keep a chat app accessible on column 1 within all contexts, for example. The anaology to `tmux` would be that rows are sessions and columns are windows.

I realized that I wanted a similar "gutter" feature in `tmux` for `org-mode` - I wanted to be able to have my emacs window in window 0 of all my sessions.

Enter `link-window` ! `tmux link-window` allows a single window to be shared across multiple sessions. It accepts `-s` a source window and `-t` a destination window, which can have a `session:` identifier prefixed to it (with the colon to separate the session and window identifier). So if I always keep a window titled `'org'` open in session 0, then I can run `tmux link-window -s '$0:org' -t '0'` in any other org session to link window 0 to the `@org` window in session 0.

https://user-images.githubusercontent.com/1221372/187304113-306ef197-52be-4446-80b8-3df5aee7f725.mp4

Using `tmux new-window` you can get a little fancier and create a helper script that'll automatically create the `@org` window in session 0 if it doesn't already exist.

```bash
#!/usr/bin/env bash
# unnoficial strict mode, note Bash<=4.3 chokes on empty arrays with set -u
# http://redsymbol.net/articles/unofficial-bash-strict-mode/
set -euo pipefail
IFS=$'\n\t'
shopt -s nullglob globstar

ORG_DIR="$HOME/org-mode"

tmux has-session -t '$0' > /dev/null \
  || (echo 'no session $0 !' && exit 1)
tmux list-windows -t '$0' | grep 'org' > /dev/null \
  || tmux new-window -d -c "$ORG_DIR" -n 'org' -t '$0:0' emacs \
  || (echo 'could not create $0:org !' && exit 1)
session_id="$(tmux display-message -p '#{session_id}')"
[[ "$session_id" == '$0' ]] && echo 'in session $0, done' && exit 0
tmux list-windows | grep 'org' > /dev/null && echo '@org already exists, done' \
  || tmux link-window -s '$0:org' -t '0' 2> /dev/null \
  || (echo 'something went wrong :(' && exit 1)
```

https://user-images.githubusercontent.com/1221372/187304494-7a74787a-4103-4a80-b96b-a39c9a75cf63.mp4

[1]: https://en.wikipedia.org/wiki/Terminal_multiplexer
[2]: https://orgmode.org/
[3]: https://www.man7.org/linux/man-pages/man1/tmux.1.html#CLIENTS_AND_SESSIONS
[4]: https://www.man7.org/linux/man-pages/man1/tmux.1.html#DEFAULT_KEY_BINDINGS
[5]: https://twitter.com/Quelklef/
[6]: https://xmonad.org/
