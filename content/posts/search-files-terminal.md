---
title: Finding files using the terminal
date: 2024-08-14
tags: development
---

Finding notes or files on your system can be hard. Where did you put that file? We can use command line tools to search for files.

One of the easiest ways to find files on your system based on their name is using `fd`.

```bash
fd [PATTERN]
```

This will return all the relevant files with your current directory. It will also ignore the patterns from `.gitignore` by default.

Another nice tool to find files is `fzf` which is fuzzy finder.

```bash
fzf
```

Will open-up an search prompt in the current directory. If you use the neovim plugin telescope `find_files` functionality it should look familiar.


## Search in file content

However if you also want to search the content of your files ripgrep is probably the right tool `rg` is probably the right tool.

```bash
rg [PATTERN]
```

These to types of file search will probably find you the correct file or content 90% of the time.

At least if you know what you're looking for.

## Links

- https://github.com/sharkdp/fd
- https://github.com/junegunn/fzf
- https://github.com/BurntSushi/ripgrep

[[tool]] [[cli]] [[unix]]
