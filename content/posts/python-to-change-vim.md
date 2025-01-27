---
title: Python to Change VIM 
date: 2024-08-22
tags:
  - vim
  - python
---

The programming language I'm most familiar with is [python](https://python.org). I also started to learn vim recently. How can I combine the two?

I discovered that I can use python (or any programming language for that matter) to manipulate text in vim. You simply need to read from the standard input, and provide a output.

## Script

So I wrote a basic python script that can lower case a string. 

`~/.dotfiles/scripts/lower`

```python
#!/usr/bin/python3
import sys


def to_lower() -> str:
    return sys.stdin.read().lower().rstrip()


if __name__ == "__main__":
    print(to_lower())
```

I then made the file executable

```bash
chmod +x ~/.dotfiles/scripts/lowercase
```

and added the script to my path. I for example added the `~/.dotfiles/scripts` directory to my path (in `~/.zshrc`), allowing me to call the executables name from my shell.

```bash
export PATH="$HOME/.dotfiles/scripts:$PATH"
```

I can now hit `!!` in vim, and type the name of my script `lower` and hit enter.

The line below my cursor is now lower cased.

## Links

[[vim]] [[python]]
