# External commands in Git

I've used git for, let's see, somewhere between 15 and 20 years, and I'm always learning new things about it.

Claude pointed out git's support for external (custom) commands the other day, and I had never come across this. These are also called "git subcommands" or "git plugins".

External commands essentially let you write little plugins for git. These plugins are simple bash scripts. Well, actually, they don't need to be bash scripts; they just need to be an executable that takes arguments.

Let's go over an example of why this would be useful.

## A git task

One common cleanup task in a repo is to clean out branches that have already been merged. Otherwise they visually clog up the list of local branches you see with `git branch`.

My previous way to do this was the command `git branch --merged | egrep -v "(^\*|main|dev)" | xargs git branch -d`. This command lists the merged branches, filters out the `main` and `dev` branches so that they are not deleted, and pipes the resulting list to `git branch -d` to delete each. But as you can see, this command is not easy to memorize.

I asked Claude to write a simple bash script that figured out what branches were already merged, list them, and prompt me if I really wanted to delete them, and then delete them if I did. The result was `git-branch-merged`. Its source code is a little long to put here, and is not super interesting anyway.

Curiously, Claude recommended putting the script in my `~/bin` folder so that I could invoke it with `git branch-merged`. And so I did and it worked. That's because if the command `git <command>` is not built in, git looks for a script called `git-command` in your PATH.

Well that's cool! I had just created a plugin for git.

## Why you'd want to do this

Okay, so what is the advantage of invoking the script through git and not directly?

One thing you get is discoverability. If you run `git help -a` you see all the commands available, from the "porcelain" built-ins to the external ones it finds in PATH, the ones that start with `git-`.

Another reason is that git's global options apply before your script starts, so that `git -C ~/some/other/repo branch-merged` works from anywhere without changing the working directory.

Git also hands the script a prepared environment with variables such as `GIT_DIR` and `GIT_PREFIX` already set, so they are available for your script to use.

Git never fails to impress me with its depth of features.