---
layout: post
title: Workflow Thoughts with Git by DAV
excerpt: "How we work with Github (by DAV)."
categories: articles, git
tags: git, github
---

---
Title: 
Layout: Default
---

## Why Git is not like SVN

Git is different to svn and I think that's where a lot of confusion stems. (Certainly where my confusion comes from...) Svn is designed around having a central repository that people checkout, make changes to, and then commit up to the centrally hosted repository. Locally checked-out repos are just that -- local copies of the repo. Git calls itself a distributed version control system, and so when you clone out a repository to your local machine, you are getting a fully-fledged copy of that repository with its own detailed versioning history, branches, merges etc.. It seems like we want to use git like svn at the moment (and git lets you do this), and there is a good tutorial of how to do this here:

[Git tutorial for 'centralised workflow'](https://www.atlassian.com/git/tutorials/comparing-workflows/centralized-workflow)

## Summary of the 'SVN-style' workflow

Here's a brief summary of the above tutorial.
 
Since we all have merge rights and work from a single repository, there's no need to fork your own branches of the code (although you could do, and it's more "Git-esque" to do so apparently). After you've cloned a copy of the repo to your computer, you open your favourite editor (vim) and hack away all morning, powered by coffee and [Tunnock's teacakes](http://www.tunnock.co.uk/products/teacakes.aspx). Then you do this:

```
git commit LSDWhiskyFlowDynamics.cpp -m "Added some changes to blended malt routines"
```

But this is git, not svn, and this will only commit your modifications to your *local* repository. If you want to push it to the central repository (i.e. the one on github) you have to tell git to push:

```
git push origin master
``` 

And if nobody has made any changes since, your changes will be added to the github repo. If someone has made changes since, you will get an error message from Git telling you something about tips of branches being behind in your local repo (due to an early frost, presumably) and you need to do a git pull. But don't just type `git pull`, instead go for:

```
git pull --rebase origin master
```

This tells git to make your commits go to the head of the master branch on the remote repo, so your changes are seen as the most up to date. `git pull` still works, but when someone else tries to push to the remote repo, their commits get mangled with a superfluous merge (which is not necessarily bad, it just looks a bit confusing when browsing the repo)

If you want to see what the differences are between your 'local' and the github repository of the project, you can do:

```
git fetch
git diff <local branch> <remote-tracking branch>
```

Git `fetch` just grabs the changes from the remote repo -- it doesn't actually merge them into your local branchm but is needed before you can run the `diff` command.

For us this is probably:

```
git diff origin origin/master
```

### Other gotchas

1. There is no such thing as `git update`. (As much as I wanted to believe there was). Do a a `git pull --rebase` instead.
2. `git add` does not commit the file just added to anything. You have to follow it with a `git commit` to get it under version control in your local repository. (And remember `git commit` only commits it locally...)
3. `git status` tells you the status of your local repository. Actually this isn't a gotcha, and is one of the few things to work like svn does. Hooray!
