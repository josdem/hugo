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

**Ignore Files**

Ignored files are usually build directories or generated files. It is a good practice ignore those files in order to do that you can simply create a file named `.gitignore` and add those files Git should ignore.

```txt
credentials.yml
.classpath
build/
*.log
**swp
```

For example:

* `credentials.yml` Is a personal file with your username and password
* `.classpath` A computer generated file
* `build/` A computer generated directory
* `*.log` An asterisk is a wildcard that matches zero or more characters
* `**swp` Double asterisk to match directories anywhere in the repository

**Log**

The simpliest way to see your repository history is using this command:

```bash
git log
```

*Output*

```bash
commit 57b3fcd5985d5a905690154ff37d83753b2a0fcc
Author: josdem <joseluis.delacruz@gmail.com>
Date:   Sat Feb 23 09:24:32 2019 -0500

    #1 Adding image

commit fc5266ceb809a161d82db3b54c906efac846c1e3
Author: josdem <joseluis.delacruz@gmail.com>
Date:   Sat Feb 23 09:10:43 2019 -0500

    #1 Changing file name

commit 33d8a5feb6329c87f525685b39ce5e9b076cd3f3
Author: josdem <joseluis.delacruz@gmail.com>
Date:   Sat Feb 23 09:07:58 2019 -0500

    #1 Deleting Chapter 1 in txt format

commit 89365cd47aee0a534f2bb65fd1ebbd1600f6d0e3
Author: josdem <joseluis.delacruz@gmail.com>
```

To see only the commits of a certain author

```bash
git log --author=josdem
```

Compressed log where each commit has a single line

```bash
git log --pretty=oneline
```

*Output*

```bash
57b3fcd5985d5a905690154ff37d83753b2a0fcc #1 Adding image
fc5266ceb809a161d82db3b54c906efac846c1e3 #1 Changing file name
33d8a5feb6329c87f525685b39ce5e9b076cd3f3 #1 Deleting Chapter 1 in txt format
89365cd47aee0a534f2bb65fd1ebbd1600f6d0e3 #1 Adding Chapter 1 file
3929d19b3183f36a2c22350b621267142d0f713d #1 Adding first line in Chapter 1
b2fefdbc93de041541b2c1cda0da26e8369a9cd8 #1 Adding Chapter 1
306b0c78b8062ed59c81d3262916d827f61a64f7 Adding Git ignore file
52d856f9c2b44352abd77487891cdfba4af0f3b9 first commit
```

*Undo Local Changes*

In case you did something wrong, you can replace local changes using this command

```bash
git checkout -- filename
```

For a single file or

```bash
git checkout .
```

To undo all changes.


[Return to the main article](/techtalk/continuous_integration_delivery)
