---
title: Local git repo for remote code syncing
description: Helpful while non of other options works
date: 2025-05-25
categories:
    - Tips
tags:
  - git
  - linux
---

There are multiple way to edit and commit file on a remote host, like VSCode, Jetbrains Sync.
But sometimes these may not work for your case. For example:

1. The host SSH-able does not connect to internet, so can't do git pull.
2. Glibc version too old for VSCode.
3. Private repo not public available.

In these cases, one way to make it work is to use a local git repo on remote host, and push to it use SSH.
(For example, below I use a private LustreSrc package I'm working on.)

On target host

```bash
mkdir -p ~/git-repos/lustreSrc.git
cd ~/git-repos/lustreSrc.git
git init --bare # this initialize the local git repo
```

On local git repo, which you should have already cloned it locally, add a remote target on host.

```bash
git remote add mdstest mdstest:/home/$USER/git-repos/lustreSrc.git
git push -u mdstest b2_15 # <branch_name> you want to work on
```

On target host

```bash
git clone ~/git-repos/lustreSrc.git
git checkout b2_15 # Checkout <branch_name> specified above
git pull
```

Then you are all set to do bi-directal sync between local and remote! Just push and commit as usual.


