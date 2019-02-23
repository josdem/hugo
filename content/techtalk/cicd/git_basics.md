+++
title =  "Git Basics"
description = "Getting started with Git"
date = "2019-02-23T07:32:32-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code","ci","cd"]
+++

In this technical post we will cover the basic and essential commands in [Git](https://en.wikipedia.org/wiki/Git). Thank you [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) for all your creations for making this developer world a best place to live.

The getting started recipe:

```bash
echo "# git-workshop" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:josdem/git-workshop.git
git push -u origin master
```

Where:

* `git init` Create a new git repository
* `git add` Add a new file
* `git commit -m` Commit changes from a single file
* `git remote add` Add a new remote repository
* `git push` Push your commits to remote repository

**Branching**

In order to develop features isolated from each other we can use branches, repositories consists of "trees" maintained by git. So you have master as main tree and you will be able to create as many branches as you want. Use branches for development and merge them back to the master upon completion.


```bash
git checkout -b feature/1
```

With previous command we created a new feature branch, usually we create a branch for each new feature we are working. So let's create a new file called `Chapter_1.md`.

```markdown
# Chapter 1

When I was six years old, I saw, once, a splendid picture.
```

Then let's add that file to our local repository

```bash
git add Chapter_1.md
gc -m "#1 Adding Chapter 1 file"
```

Now let's add more content to our file

```markdown
# Chapter 1

When I was six years old, I saw, once, a splendid picture, in a book about the virgin forest called *Stories of Life*. It represented a boa snake that swallowed a wildcat. This is the copy of the drawing.

![Snake Swallowed a Wildcat](/img/snake_swallowed_wildcat.png)
```

And then other commit

```bash
git commit -m "#1 Completing first paragraph"
```

Here we are adding "#1 ..." as feature reference. You can continue adding and committing changes with descriptive comments, try to do small changes for each commit so in that way will be easier to do code reviews. Once you complete your feature you can merge that changes to master.

```bash
git checkout master
git merge feature/1
git push origin master
```

**Checkout**

You can create a fresh copy of a local repository by running this command

```bash
git clone git@github.com:josdem/git-workshop.git
```

At this point you have only master, you can see that with this command:

```bash
git branch
```

*Output*

```bash
* master
(END)
```

In order to update remote tracking branches you can use this command

```bash
git fetch
```

Then you can use this command to move to the fresh copy of `feature/1` branch

```bash
git checkout feature/1
```

*Output*

```bash
Branch 'feature/1' set up to track remote branch 'feature/1' from 'origin'.
Switched to a new branch 'feature/1'
```

Now we should have two local branches ready

```bash
git branch
```

*Output*

```bash
* feature/1
  master
(END)
```

**Delete a branch**

You can delete a local branch with this command

```bash
git branch -d feature/1
```

If you want to delete a remote branch you can use this command

```bash
git push origin --delete feature/1
```

**Pull**

To incorporate changes from a remote repository into the local branch use this command

```bash
git pull origin branch_name
```

That's it `git pull`, updates your current local branch with the latest changes from the remote server. `git fetch` never change or update any of your own local branches, it a is safe operation.

**Log**

The simpliest way to see your repository history is using this command:

```bash
git log
```

[Return to the main article](/techtalk/continuous_integration_delivery)
