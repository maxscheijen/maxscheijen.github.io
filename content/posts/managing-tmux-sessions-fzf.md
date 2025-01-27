---
title: Managing tmux sessions with fzf
date: 2024-12-25
tags: setup 
---

Terminal multiplexers like `tmux` are essential tools for developers, allowing us to manage multiple terminal sessions efficiently. While `tmux` itself is powerful, we can enhance its functionality by combining it with `fzf`, the fuzzy finder, to create a more intuitive session management workflow.

## The Basic Approach

The foundation of this solution lies in combining `tmux`'s session listing capability with `fzf`'s search interface. Here's how we can list all sessions using `fzf`:

```sh
tmux ls | awk -F: '{print $1}' | fzf
```

This command breaks down into three parts:

- `tmux ls` lists all active sessions
- `awk -F:` splits the output at the colon and extracts session names
- `fzf` provides an interactive search interface

## Creating a Session Switcher

To make this truly useful, we can create a script that not only lists sessions but also switches to the selected one. Here's the basic implementation:

```sh
session=$(mux ls | awk -F: '{print $1}' | fzf)
if [ -n "$session" ]; then
    tmux switch-client -t "$session"
else
    echo "No session selected."
fi
```

## Enhanced Version with Visual Indicators

To make the interface more informative, we can add a colored indicator for the currently active session. Here's the improved version:

```sh
session=$(tmux ls | awk -F: '
    /attached/ {print $1 "\033[32m *\033[0m"}
    !/attached/ {print $1} 
' | fzf --ansi)
session=$(echo "$session" | sed 's/ (attached)$//')
if [ -n "$session" ]; then
    tmux switch-client -t "$session"
else
    echo "No session selected."
fi
```

This enhanced version uses green asterisks to highlight attached sessions, making it immediately clear which session is currently active.

## Integration with `tmux`

To make this functionality easily accessible, add this line to your `.tmux.conf`:

```sh
bind-key t run-shell "tmux neww $HOME/path/to/script"
```

Now you can trigger the session switcher using the `prefix + t` key combination.

Remember to make your script executable with `chmod +x path/to/script` before using it.

This simple yet powerful integration demonstrates how combining familiar tools in thoughtful ways can significantly improve our development workflow. By leveraging `fzf`'s search capabilities with `tmux`'s session management, we've created a more intuitive way to handle multiple terminal sessions.

## Links

[[tmux]] [[fzf]]

