---
title: Finding files using the terminal
date: 2024-08-14
tags:
    - cli
    - tool
    - productivity
---

Finding notes or files on your system can be hard. Where did you put that file? We can use command line tools to search for files.

One of the easiest ways to find files on your system based on their name is using `fd`.

```shell
fd [PATTERN]
```

This will return all the relevant files with your current directory. It wil also ignore the patterns from `.gitignore` by default.

Another nice tool to find files is `fzf` which is fuzzy finder.

```shell
fzf
```

Will open-up an search prompt in the current directory. If you use the neovim plugin telescope `find_files` funcionality it should look familiar.


## Search in file content

However if you also want to search the content of your files ripgrep is probabuly the right tool `rg` is probably the right tool.

```shell
rg [PATTERN]
```

These to types of file search will probably find you the correct file or content 90% of the time.

Atleast if you know what you're looking for.

## Links

- https://github.com/sharkdp/fd
- https://github.com/junegunn/fzf
- https://github.com/BurntSushi/ripgrep
