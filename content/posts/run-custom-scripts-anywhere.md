---
title: Run Custom Scripts From Anywhere
date: 2024-08-15
tags: development
---

Lately I started to look into creating simple scripts that can automate some tasks for me. So I started looking into shell scripting.

I wanted to run and have access to these scripts from anywhere in my terminal. After watching a lot of [Mischa van den Burg](https://mischavandenburg.com/) I started taking in interest in more thorough note taking.

My first script would be for easier note taking.

So I took his [blog script](https://github.com/mischavandenburg/dotfiles/blob/main/scripts/blog) tweaked it a little, and wanted to make it executable from anywhere in my terminal.

`~/.dotfiles/scripts/note`

```bash 
#!/bin/bash

# Get the filename
read -p "Enter a filename: " filename

# Get the current date
date=$(date +"%Y-%m-%d")

# Set the second brain directory
directory="$SECONDBRAIN/0-inbox/"

# Create markdown name based on filename
touch "$directory/$filename.md"

# Add formatting 
echo "---" >> "$directory/$filename.md"
echo -e "title: " >> "$directory/$filename.md"
echo "date: $date" >> "$directory/$filename.md"
echo "tags:" >> "$directory/$filename.md"
echo -e "  -" >>"$directory/$filename.md"
echo "---" >> "$directory/$filename.md"

echo -en "\n" >> "$directory/$filename.md"
echo -en "\n" >> "$directory/$filename.md"
echo -en "\n" >> "$directory/$filename.md"

echo "## Links" >>  "$directory/$filename.md"
echo -en "\n" >> "$directory/$filename.md"

# Open up in NeoVim
nvim '+ normal 2GzzA' "$directory/$filename.md"
```

I ensured that every script in `~/.dotfiles/scripts/` was executable by running

```bash
chmod +x ~/.dotfiles/scripts/*
```

After this I added the scripts directory to the PATH by adding the following line to my `~/.zshrc` file

```bash
export PATH="$HOME/.dotfiles/scripts:$PATH"
```

Reload my `~/.zshrc` by running

```bash
exec ${SHELL} -l
```

And now I'm able to run

```bash
note
```

creating a note with formatting in a specific directory.

## Links

https://github.com/mischavandenburg/dotfiles/tree/main/scripts

[[cli]] [[unix]] [[bash]]
