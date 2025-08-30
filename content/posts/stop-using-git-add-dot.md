---
title: Stop using "git add ."
date: 2025-08-16
---

We’ve all done it, `git add .`, adding untracked files to the staging area: `.env`, `TODO`, or who knows what, and then committing them.

Stop doing that. Instead, use:

```sh
git add -u
```

This stages only the files that are already tracked by Git.

The `-u` (short for update) flag tells Git to stage only modifications and deletions of already tracked files. It will not stage new, untracked files.

This makes it much harder to accidentally commit unwanted files, while still capturing all changes to the files that are already part of your repository.

If you want to stage only modified files and leave deletions out, you can use:

```sh
git add -u :/
```

And if you need finer control, consider git add `-p`, which lets you review and selectively stage changes hunk by hunk.
