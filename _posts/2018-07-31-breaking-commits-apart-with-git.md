---
layout: post
title: Breaking Commits Apart With Git
summary:
  I like the code in my commits to be related, but sometimes an unrelated change
  will slip past my guard. In cases like this, it can be extremely helpful to
  know how to split a given commit apart.
author: Tom Schaefer
author_url: http://teecom.com/people/tommy-schaefer/
tags: git
---

When working on a new feature, I try to group my code under logical commit
boundaries. That is to say, I like the code in my commits to be related.
Sometimes though, an unrelated change will sneak past my guard. For this reason,
I find it helpful to know how to break one large commit into several smaller
ones.

A word of caution, however. This technique can be dangerous when applied to
branches shared by multiple contributors. To break one commit into several is
to rewrite history. Rewriting history can cause headaches for other contributors
on a branch, leading to merge conflicts or even loss of work. Because of this,
it is only ever safe to rewrite history on branches with no other collaborators.
Even though I usually work alone on my projects, I tend to work in feature
branches for this reason. Just to be safe.

## Rebasing

To break a commit apart, we will use the `edit` command in an interactive
rebase. To start an interactive rebase, use `git rebase -i` with some kind of
reference. This reference is any pointer to a commit which you would like to
rebase against. It could be the root commit, a particular hash, or even a branch
name.

```
$ git rebase -i --root

$ git rebase -i 133b47e

$ git rebase -i origin/master
```

Running one of the variants above will open something like this in a text
editor:

```
pick 133b47e The first commit message
pick 6468919 Another commit message
pick e1d4848 The last commit message


# Rebase f73e4e2..e1d4848 onto f73e4e2 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

## Breaking the commit apart

Find the commit that you would like to break apart and change `pick` to `edit`.
Save and quit to continue. You should see output like this:

```
Stopped at 6468919... Add "breaking-a-commit-apart" and "other" files
You can amend the commit now, with

		git commit --amend

Once you are satisfied with your changes, run

		git rebase --continue
```

At this point, we've traveled through Git's commit history and arrived at a
target commit. To unstage the relevant changes, run:

```
git reset HEAD^
```

From here, you are able to `add` changes and `commit` them using your normal
process. When you're satisfied with the more granular commits that you've made,
you can run:

```
git rebase --continue
```

This will replay the commits we traversed on top of your modified history. The
end result is a commit history where your large commit has been broken apart.

## Pushing Your Changes

If this is the first time you're pushing a set of commits upstream, your
`git push` should work as expected. If, however, your changes modify a commit
that the upstream branch is aware of, you're going to need to force push.

I recommend using this variant of the `--force` flag:

```
git push --force-with-lease
```

Git's `--force-with-lease` flag protects you from overwriting upstream changes.
If, for example, another contributor commits to your feature branch, your push
will fail rather than overwrite their work.
