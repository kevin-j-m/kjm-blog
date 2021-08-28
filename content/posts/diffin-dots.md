+++
title = "Diffin' Dots"
date = 2019-12-30T21:04:53-05:00
tags = ["git"]
categories = []
summary = "Git comparisons of the future...today!"
description = "A comparison of ways to, well, compare, different objects in git."
+++

## A Sweet Surprise

Recently, a project needed to re-order some commits on a git branch. We
initially performed this interactive rebase on a separate branch, so we didn't
make a mistake on the original branch. After performing that rebase, we then
used Github's [compare view](https://help.github.com/en/github/committing-changes-to-your-project/comparing-commits-across-time#comparing-branches), expecting to see no files changed.

Instead, what we saw was every change that was introduced in all of the commits
that were reordered on the new branch. This very much surprised us. We made no
functional changes and were expecting to use this comparison as proof that we
didn't accidentally drop a commit or do something terrible on accident.

We then looked to compare the branches locally. When we did that, we saw no
diff.

```bash
⇒ diff-test|reorder git diff master

(END)
```

Did we find a bug in Github? Did our re-ordering work? At this point, we had no
idea.

## Read The Recipe

We dug into the documentation for `diff`, a tool I use frequently,
but don't spend much time considering how it works.

Let's read some of the help on how to use the diff tool:

```bash
⇒ git diff --help
GIT-DIFF(1)                                      Git Manual                                      GIT-DIFF(1)
...
       git diff [<options>] <commit> [--] [<path>...]
           This form is to view the changes you have in your working tree relative to the named <commit>.
           You can use HEAD to compare it with the latest commit, or a branch name to compare with the tip
           of a different branch.

       git diff [<options>] <commit> <commit> [--] [<path>...]
           This is to view the changes between two arbitrary <commit>.

       git diff [<options>] <commit>..<commit> [--] [<path>...]
           This is synonymous to the previous form. If <commit> on one side is omitted, it will have the
           same effect as using HEAD instead.

       git diff [<options>] <commit>...<commit> [--] [<path>...]
           This form is to view the changes on the branch containing and up to the second <commit>, starting
           at a common ancestor of both <commit>.
```

Almost exclusively I've used diff in the form of `git diff other-ref`. And
with that, I would see what, if any, file changes occurred between those two
branches or commits. This is the two-dot comparison. However, Github uses the
[three-dot comparison](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-comparing-branches-in-pull-requests#three-dot-and-two-dot-git-diff-comparisons)
by default.

In this scenario, the common ancestor is the latest commit on the reordering
branch that wasn't reordered. After the rebase, all of those reordered commits
have different SHAs, so are seen as different commits. As such, using the
three-dot option, git considers all of those SHAs as different or new, so all of
them show as changes, even though there are no differences in the file contents
themselves.

## Homemade Flavor

Let's look at demonstrating this behavior entirely locally.

### Setup

First, let's create a new repository:

```bash
⇒ mkdir diff-test
⇒ cd diff-test
diff-test|⇒ git init .
Initialized empty Git repository
```

We'll start by adding a few new files, and making the first commit:

```bash
diff-test|master ⇒ echo "Creamy Vanilla Ice Cream done Dippin’ Dots style" > vanilla.txt
diff-test|master⚡ ⇒ echo "Creamy Milk Chocolate Ice Cream.  Someone pass the spoon" > chocolate.txt
diff-test|master⚡ ⇒ git add --all
diff-test|master⚡ ⇒ git commit -m "IFO: initial flavor offering"
```

We will then add two more commits with some additional changes:

```bash
diff-test|master ⇒ echo "Dippin' Dots Strawberry Ice Cream is as sweet as a summer's day and so berry, berry good" > strawberry.txt
diff-test|master⚡ ⇒ git add --all
diff-test|master⚡ ⇒ git commit -m "Introducing strawberry"

diff-test|master ⇒ echo "Orange Flavored Ice" > orange.txt
diff-test|master⚡ ⇒ git add --all
diff-test|master⚡ ⇒ git commit -m "Introducing orange"
```

### Introduce a change

At this point, we realize that it's important to us that orange be introduced in
our history before strawberry.

Let's create a new branch to address this, and look at the current order of the
commits:

```bash
diff-test|master ⇒ git checkout -b reorder
Switched to a new branch 'reorder'
diff-test|reorder ⇒ git log --oneline

e378256 (HEAD -> reorder, master) Introducing orange
f455e5b Introducing strawberry
e42566b IFO: initial flavor offering
```

We will then perform an interactive rebase, going back two commits in history.
During that rebase, we will switch the order of orange and strawberry, so orange
is first, as we need it:

```bash
diff-test|reorder ⇒ git rebase -i HEAD~2

pick e378256 Introducing orange
pick f455e5b Introducing strawberry
```

After performing the rebase, here is the order of our commits:

```bash
diff-test|reorder ⇒ git log --oneline

5fa36f4 (HEAD -> reorder) Introducing strawberry
7a1d500 Introducing orange
e42566b IFO: initial flavor offering
```

Congratulations! We've done what we set out to do, performing a rebase to
reorder our commits. Now let's compare that to our original branch to make sure
everything looks to be as we'd expect.

### `diff`erences

We start by comparing this branch to the original branch:

```bash
diff-test|reorder ⇒ git diff master

(END)
```

We see that there's no difference, which we're expecting and hoping for. No
files changed, no functionality is different; only the history has changed.

If we diff using the two-dot version, we see the same result:

```bash
diff-test|reorder ⇒ git diff master..reorder

(END)
```

That sounds right, because according to the documentation, the two-dot usage is
the same as not providing dots:

```bash
 git diff [<options>] <commit>..<commit> [--] [<path>...]
          This is synonymous to the previous form. If <commit> on one side is omitted, it will have the
          same effect as using HEAD instead.
```

Lastly, let's perform a three-dot diff:

```bash
diff-test|reorder ⇒ git diff master...reorder

diff --git a/orange.txt b/orange.txt
new file mode 100644
index 0000000..09d8732
--- /dev/null
+++ b/orange.txt
@@ -0,0 +1 @@
+Orange Flavored Ice
diff --git a/strawberry.txt b/strawberry.txt
new file mode 100644
index 0000000..6ab358d
--- /dev/null
+++ b/strawberry.txt
@@ -0,0 +1 @@
+Dippin' Dots Strawberry Ice Cream is as sweet as a summer's day and so berry, berry good
(END)
```

Here is where we see that it looks like each of those files have been changed as
a result of this rebase. However, the files themselves __haven't__ changed. What
has is the commits themselves. Their common ancestor is the initial commit of
the repository, so everything that happened since then shows as a difference.

## The Cherry On Top

Locally, unless you tell git otherwise, `diff` will show you the file change
differences between your comparisons. Github by default will show you the
changes from the common ancestor between what's changed.

Most of the time, this different behavior won't or shouldn't matter in the
course of your workflow. However, if you're doing something a bit more
adventurous, or perhaps ill-advised, knowing how git will, by default, compare
changes locally and how Github, by default, will surface those changes can be
paramount.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/diffin-dots).
