---
title: Supercharge Your Git History Navigation with Fuzzy Find
date: 2024-11-11
tags: setup
---

Ever found yourself scrolling endlessly through `git log` trying to find that one specific commit? Let's explore how to transform this tedious process into a smooth, interactive experience using `fzf`, a powerful command-line fuzzy finder. This approach will not only make navigating Git history more efficient but also more enjoyable.

## Basic Setup: Your First Fuzzy Git log

The simplest way to start is by piping your git log into `fzf`:

```sh
git log --oneline | fzf
```

This basic command already gives you an interactive interface where you can type to filter commits instantly. But we can do much better.

## Adding Rick Preview

What if you could see the actual changes while browsing through commits? We can achieve this by adding a preview window:

```sh
git log --oneline | fzf --preview="git show {1}"
```

The magic here is in the `--preview` flag:

* `{1}` refers to the first "field" in each line (the commit hash)
* As you move through commits, `fzf` automatically shows the full diff in the preview window


## Quick Commit Hash Copying 

Often, you'll want to grab a commit hash to use it in another command. Let's enhance our command to automatically copy the selected commit hash to the clipboard:

```sh
git log --oneline | fzf --preview="git show {1}" | awk '{print $1}' | pbcopy
```
This pipeline:

1. Shows the interactive commit list
2. Lets you select a commit
3. Extracts just the hash with `awk`
4. Copies it to your clipboard (`pbcopy` for macOS, use `xclip -selection clipboard` for Linux)

## Making It Beautiful: Adding Color and Context

Let's enhance the visual experience by adding colors and more contextual information:

```sh
git log \
  --color=always \
  --date=short \
  --pretty=format:"%cd %C(auto)%h %d %s %C(green)%cr %C(bold blue)<%an>" | \
fzf \
  --ansi \
  --preview "git show --color=always {2}" | \
awk '{print $2}' | pbcopy
```

This improved version includes:

1. Commit date
2. Hash
3. Branch/tag decorations
4. Commit message
5, Relative time
6. Author name
7. Syntax-highlighted diffs in the preview

Note that we use `{2}` instead of `{1}` here because the commit hash is now in the second column of our formatted output.

## The Ultimate Solution: A Complete Script

Here's a production-ready script that handles edge cases and provides a polished experience.

 1. Check for `fzf` installation
 2. Verifies you're in a Git repository
 3. Gracefully handles clipboard operations across platforms.

This script can then be added to your system's PATH for convenient access from any directory. See how to do so [here](run-custom-scripts-anywhere).

```sh
#!/bin/bash

# Check if fzf is installed
if ! command -v fzf &> /dev/null
then
    echo "fzf could not be found. Please install fzf before running this script."
    exit 1
fi

# Check if the current directory is a Git repository
if ! git rev-parse --is-inside-work-tree &> /dev/null
then
    echo "Current directory is not a Git repository."
    exit 1
fi

git_log=$(
  git log \
  --color=always \
  --date=short \
  --pretty=format:"%cd %C(auto)%h %d %s %C(green)%cr %C(bold blue)<%an>"
)

# Use fzf to select a commit from the Git log
selected_commit=$(
  echo "$git_log" | \
  fzf \
  --layout=reverse \
  --border \
  --border-label="Fuzzy Find Git Log" \
  --info=default \
  --ansi \
  --preview "git show --color=always {2}" \
  --preview-label='[ Diff ]' \
  --header 'Copies commit hash to clipboard | CTRLC to cancel' \
  --header-first \
  --pointer='→' \
)

# Exit when if no commit is selected 
if [ -z "$selected_commit" ]; then
  exit 0
fi

# Extract the commit hash from the selected line
commit_hash=$(echo "$selected_commit" | awk '{print $2}')

# Copy the commit hash to the clipboard (Linux and macOS)
if [ "$(uname)" == "Darwin" ]; then
    echo -n "$commit_hash" | pbcopy
else
    echo -n "$commit_hash" | xclip -selection clipboard
fi

echo "Copied commit hash '$commit_hash' to clipboard."
```


## Links

[[fzf]] [[cli]] [[git]]
