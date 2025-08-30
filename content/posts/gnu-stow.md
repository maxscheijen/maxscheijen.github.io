---
title: How I manage my dotfiles
date: 2024-11-28
---

I store all my dotfiles in a dedicated git repository called `dotfiles` with a remote on GitHub. This allows me to track changes, and clone and pull them to other machines.

I then use `stow` to create **symlinks** to place the dotfiles in the appropriate places.

This creates symlinks from for example `~/dotfiles/.zshrc` to `~/.zshrc`.

```bash
stow zsh
```

I organize them by tool.

```
dotfiles/
├── zsh/
│   └── .zshrc
├── git/
│   └── .gitconfig
└── README.md
```

Setup a machine using `dotfiles.sh`.

```bash
#!/bin/bash
sudo apt install git zsh
git clone git@github.com/maxscheijen/dotfiles $HOME/dotfiles
stow git zsh
```

...
