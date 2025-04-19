---
title: Fuzzy Finding Through Your Command History
date: 2024-11-07
---

We've all been there: you type a complex command in your terminal, and a few weeks later, you need it again. Even with autosuggestion enabled, remembering these commands can be tricky. Wouldn't it be nice to quickly search through your terminal history using fuzzy finding?

Let's build a simple tool that lets you fuzzy find through your command history, select a command, and execute it. We'll use `fzf` as our fuzzy finder of choice.

## Basic Implementation

The simplest approach is to pipe your history directly to `fzf`:

```sh
history | fzf
```

However, you'll quickly notice this gives us lots of duplicate commands, and each line includes some unnecessary numbers from the history format. Let's clean that up!

To remove the leading whitespace, numbers, and timestamps, we can use `sed` combined with `sort` and `uniq` to get a clean, sorted list of unique commands:

```sh
sed 's/^[[:space:]]*[0-9]*[[:space:]]*//; s/^: [0-9:]*;//' | sort | uniq
```

Now we can combine these pieces:

```sh
history | sed 's/^[[:space:]]*[0-9]*[[:space:]]*//; s/^: [0-9:]*;//' | sort | uniq | fzf
```

To actually execute the selected command, we just need to pipe it to `sh`:

```sh
history | sed 's/^[[:space:]]*[0-9]*[[:space:]]*//; s/^: [0-9:]*;//' | sort | uniq | fzf | sh
```

## Enhanced Script

While the one-liner works, let's make it more robust. Here's a script that handles both `bash` and `zsh` histories, and includes some error checking:

```sh
#!/bin/bash

# Check if fzf is installed
if ! command -v fzf >/dev/null 2>&1; then
    echo "Error: fzf is not installed. Please install it first."
    exit 1
fi

# Function to read zsh history
read_zsh_history() {
    # Use fc -l with zsh to get readable history
    if [ -n "$ZSH_VERSION" ]; then
        # This needs to be run in zsh
        zsh -c 'fc -l 1'
    elif [ -r ~/.zsh_history ]; then
        # Attempt to read zsh history file directly and decode it
        command -v strings >/dev/null 2>&1 && strings ~/.zsh_history
    fi
}

# Function to read bash history
read_bash_history() {
    if [ -r ~/.bash_history ]; then
        cat ~/.bash_history
    fi
}

# Function to execute the history search
search_and_execute_history() {
    # Combine both histories, with zsh history taking precedence
    (read_zsh_history; read_bash_history) |
    sed 's/^[[:space:]]*[0-9]*[[:space:]]*//; s/^: [0-9:]*;//' |
    sort |
    uniq |
    fzf --border --tac --reverse |
    (read command && eval "$command")
}

# Execute the function
search_and_execute_history
```

## Using the Script

1. Save the script as `hist` and make it executable:

```sh
chmod +x hist
```

2. Move it to your scripts directory:

```sh
mv hist ~/.dotfiles/scripts/
```

3. Add the scripts directory to your PATH in your shell configuration:

```sh
export PATH="$HOME/.dotfiles/scripts:$PATH"
```

Now you can simply type `hist` anywhere in your terminal to fuzzy find through your command history!

## Links

[[fzf]] [[cli]]
