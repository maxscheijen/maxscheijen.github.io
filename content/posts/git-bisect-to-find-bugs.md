---
title: Git Bisect to Find Bugs 
date: 2024-09-04
---

Say you and your team are working on a project. Somewhere in the last 300 commits a bug was introduced.

You searched through the commit and their messages. But you can't find which commit introduced the bug. Damm what are we going to do...

This when you can use `git bisect`, and there are only two things you need

1. the commits are ordered by time 
2. you need to know a commit where the issues was not present.

Bisect uses **binary search** to narrow down the search field, to find which commit introduced the bug.

## Example

Lets say that you know that commit `A` doesn't contain the bug (all tests pass). But in last commit `E` the tests are not passing.

However, we don't know in which commit between `A` and `E` the bug was introduced.

```
---------------------------------
|                               |
A ---------- unknown ---------- E
```

To narrow down our search, we're going to select the middle commit between `A` and `E`, commit `C`.

```
---------------------------------
|                               |
A -- unknown -- C -- unknown -- E
```

We run our tests against commit `C`. If this commit turns out to be passing than we know the faulty commit was introduced between `C` and `E` and the commits between `A` and `C` are good.


```
----------------------------------
|                                |
A --- good --- C --- unknown --- E
```

However if the bug is still present in `C` and our tests fail, than we know for sure that the commits between `C` and `E` aren't good. 

```
---------------------------------
|                               |
A --- unknown --- C --- bad --- E
```

When then repeat this process until we find the commit where are the bug was introduced.

## Basics of `git bisect` 

Git provides tools to perform this search. So you don't have to know the middle commit.

1. Start bisect

```
git start
```

2. Set the known bad commit, when no commit provided it uses the current one.

```
git bisect bad 
```

3. Set the known good commit, in this case at a tag.

```
git bisect good v1.0.0
```

4. Then we test our code.

```
python -m pytest
```

5. Depending on test turnout.

```
git bisect <good | bad>
```

6. Back to step 4 until we find the bug.


## Automatic `bisect`

Repeat step 1 till 3 as before and then run.

```
git bisect run <cmd>
```

Where `<cmd>` is the (test) command you want to run.

```
git bisect run python -m pytest 
```


## Links

https://git-scm.com/docs/git-bisect

[[git]]

