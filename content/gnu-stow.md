---
title: I Really Like GNU Stow 
date: 2024-11-28
tags: setup
---

Ever found yourself painfully recreating config files when setting up a new dev environment? Been there, done that. Let me introduce you to GNU Stow, the symlink cli that'll make your dotfiles management a easier.

## What's GNU Stow

Stow is basically a symlink farm manager that helps you organize your dotfiles by keeping them in a separate directory while magically creating symbolic links to their intended locations. It's like having a personal assistant for your config files.

### Why I use Stow for Dotfiles

1. **Centralized Management**: All my dotfiles live in one tidy, version-controlled directory (`~/.dotfiles`). No more config chaos!
2. **Crazy Good Organization**: Stow lets me separate configs by program or however I want. Total flexibility.
3. **Deployment Made Easy**: When I need to set up a new machine, it's as simple as pulling my configs and running stow. No more manual copy-pasting.
4. **Pick and Choose**: With Stow packages, I can selectively install exactly the configurations I want.

## Getting Started: The Stow Setup 

**1. Install GNU Stow**

```sh
# macOS
brew install stow

# Ubuntu/Debian
apt install stow
```

**2. Create Your Dotfiles Directory**

```sh
mkdir ~/.dotfiles
cd ~/.dotfiles
```

**3. Organize Your Configs as Packages**

I like creating packages base on programs

```sh
mkdir -p nvim/.config/nvim kitty/.config/kitty zsh
````

4. Migrate Your Configs 

Move your existing config files, keeping the same home directory structure:

<!--TODO: Check if moving works -->

```sh
mv ~/.config/nvim/* ~/.dotfiles/nvim/.config/nvim/
mv ~/.config/kitty/* ~/.dotfiles/kitty/.config/kitty/
mv ~/.zshrc ~/.dotfiles/zsh/
```

## Stow in Action 

Once your files are organized, using Stow is straightforward:

```sh
cd ~/.dotfiles

# Creates the symlink for ~/.config/nvim
stow nvim
```

In you run for example `ls -la .zshrc` you will see that it it symlinked to `~/.dotfiles/zsh/zshrc`

```sh
# Displaying symlinks for dotfiles (example ~/.zshrc)
ls -la ~/.zshrc

# output
~/.zshrc        --> ~/.dotfiles/zsh/.zshrc

# dotfile           GNU Stow
~/.config/nvim  --> ~/.dotfiles/nvim/.config/nvim
~/.config/kitty --> ~/.dotfiles/kitty/.config/kitty
```

## Best Practices

**1. Use version control**

Because all your dotfiles are in the same directory you can have just one repository for your dotfiles that has version control

```sh
cd ~/.dotfiles
git init
git add -am "Initial dotfiles commit"
```

**2. Create a setup script**

Want to automate stowing everything? Here's a quick script:

```bash
#!/bin/bash
cd ~/.dotfiles
for dir in */; do
    stow "${dir%/}"
done
```

## Common Issues and Solutions

**1. Existing Files**

Stow wont overwrite existing files that are not managed by it self. You should backup and remove existing files before stowing. You can also use `stow --adopt` to incorporate existing files (however use it with caution).

**2. Nested Directories**

Stow create parent directories as needed. You should keep the same directory structure as in your home directory.

For example the kitty config is located in `~/.config/kitty/`. So our stow setup should mirror the same structure. In `~/.dotfiles/kitty`. Where `kitty` is the stow package name. So inside this directory you should mirror `.config/kitty` resulting in `~/.dotfiles/kitty/.config/kitty`.

## Links

[[linux]] [[stow]] [[dotfiles]]
