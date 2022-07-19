---
title: Git Submodules
layout: post
date: '2022-07-19 16:55:00'
image: assets/images/markdown.jpg
headerImage: false
tag:
- Git
star: true
category: blog
author: joseph
description: knowledge of Git
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

**Starting with Submodules**

Let’s start by adding an existing Git repository as a submodule of the repository that we’re working on.

```
git submodule add https://github.com/josephchenhk/qtrader
```

Afrter this, you should notice the new .gitmodules file. This is a configuration file that stores the mapping between the project’s URL and the local subdirectory you’ve pulled it into:

```
[submodule "qtrader"]
	path = qtrader
	url = https://github.com/josephchenhk/qtrader
```

When you commit, you see something like this:

```
$ git commit -am 'Add qtrader module'
[master fb9093c] Add qtrader module
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 qtrader
 ```
Notice the 160000 mode for the qtrader entry. That is a special mode in Git that basically means you’re recording a commit as a directory entry rather than a subdirectory or a file.

**Cloning a Project with Submodules**

When you clone such a project with a submodule, by default you get the directories that contain submodules, but none of the files within them yet:

```
$ git clone https://github.com/josephchenhk/qtrader_strategies
```

The `qtrader` directory is there, but empty. You must run two commands: `git submodule init` to initialize your local configuration file, and `git submodule update` to fetch all the data from that project and check out the appropriate commit listed in your superproject:

```
$ git submodule init
$ git submodule update
```

Alternatively, you can pass `--recurse-submodules` to the `git clone` command, it will automatically initialize and update each submodule in the repository, including nested submodules if any of the submodules in the repository have submodules themselves.

```
$ git clone --recurse-submodules https://github.com/josephchenhk/qtrader_strategies
```

If you already cloned the project and forgot `--recurse-submodules`, you can combine the `git submodule init` and `git submodule update` steps by running `git submodule update --init`. To also initialize, fetch and checkout any nested submodules, you can use the foolproof `git submodule update --init --recursive`.

**Working on a Project with Submodules**

Now we have a copy of a project with submodules in it and will collaborate with our teammates on both the main project (e.g. `qtrader_strategies`) and the submodule project (e.g. `qtrader`).

*Pulling in Upstream Changes from the Submodule Remote*

1. Simply go into the directory and run `git fetch` and `git merge` the upstream branch to update the local code.
2. Alternatively, you can run `git submodule update --remote`, and Git will go into your submodules and fetch and update for you. By default the master/main branch is tracked and updated, if you want to specify a different branch `dev`, you can config in `.gitmodules` file: `git config -f .gitmodules submodule.qtrader.branch dev` (in this way, everyone else also tracks it)

*Pulling Upstream Changes from the Project Remote*

Simply executing `git pull` to get your newly committed changes is not enough. To finalize the update, you need to run `git submodule update`:

```
$ git pull
$ git status

$ git submodule update --init --recursive
$ git status
```

$\color{blue}{\textbf{Note}}$ that to be on the safe side, you should run `git submodule update` with the `--init` flag in case the qtrader_strategies commits you just pulled added new submodules, and with the `--recursive` flag if any submodules have nested submodules.

**Working on a Submodule**

Usually we want to work on both the main project and the submodule(s). 

First of all, let’s go into our submodule directory and check out a branch.

```
$ cd qtrader/
$ git checkout dev
Switched to branch 'dev'

$ cd ..
$ git submodule update --remote --merge (or --rebase)
```

If you forget the `--rebase` or `--merge`, Git will just update the submodule to whatever is on the server and reset your project to a detached HEAD state.

**Publishing Submodule Changes**

The git push command takes the `--recurse-submodules` argument which can be set to either "check" or "on-demand". The "check" option will make push simply fail if any of the committed submodule changes haven’t been pushed.

```
$ git push --recurse-submodules=check
```

If you want the check behavior to happen for all pushes, you can make this behavior the default by doing `git config push.recurseSubmodules check`.

The other option is to use the "on-demand" value, which will try to do this for you (equivalent to manully git push in each submodule).


**Further reference**

1. [Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
