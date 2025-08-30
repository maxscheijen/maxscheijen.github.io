---
title: Find and Replace in Vim
date: 2024-08-16
---

One of my goals for the remaining part of 2024 is to learn and get comfortable with vim. Both with motions and the editor (NeoVim).

I started to work with vim about 2 weeks ago and there is so much to learn. However, I think I have some of the basics covered.

One thing I wanted to learn this week is find and replace. This is an operation I use a lot, both in my coding work as in my writing. Or at least the basics of find and replace.

## Pattern

In vim you can use `:substitute` (`:s`) to find and replace text.

It follows the following pattern:

```vim
:[range]s/{pattern}/{string}/[flags][count]
```

The command searches each line in `[range]` for `{pattern}` and replaces it with `{string}`. Count is the number of times to repeat this action

When no `[range]` or `[count]` is provided, the replace only happens on the current line. 

## Basics

These are the command I'm currently using.

When I want to replace the first occurrence of “foo” with “bar” on current line. 

```vim
:s/foo/bar
```

Replace every occurrence of “foo” with bar on current line.

```vim
:s/foo/bar/g
```

One thing I do often is replace every occurrence of a pattern. The `%` character will allow you to do this for the entire file. 

```vim
:%s/foo/bar/g
```

If confirmation for substitutions is important add `i` to the end of the command.

```vim
:s/foo/bar/gc
```

The `:s` command searches for string matches. If you want to match whole words and not part of words, put replace between `\<\>`

```vim
:s/\<foor\>/bar/:w
```

By adding a `i` add the end, if you want to ignore character case.

```vim
:s/foo/bar/gi
```

https://linuxize.com/post/vim-find-replace/

