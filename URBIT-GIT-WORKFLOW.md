# ~dirwex-dosrev urbit git workflow

Copyright (c) 2015 John Franklin (jfranklin9000@gmail.com)

Licensed under the MIT license.

## The Goal

Create long lived branches on your github urbit repository fork with
your commits at the tips of the branches.

You want to create
`https://github.com/GITHUB-USER/urbit/tree/NAMED-BRANCH`
and keep it up to date with
`https://github.com/urbit/urbit/tree/SOURCE-BRANCH`
with your commits at the tip of the branch.

Example:

    GITHUB-USER = jfranklin9000
    SOURCE-BRANCH = master
    NAMED-BRANCH = nock-decrement-opcode

### Notes

This document uses hardwired paths.

This is where everything lives:

    ~/URBIT

This is where branch `NAMED-BRANCH` lives:

    ~/URBIT/urbit-NAMED-BRANCH

You can adjust these as you see fit.

This document assumes all pulls don't require manual conflict resolution.
Standard merge techniques apply if that's not the case.

The method of achieving The Goal uses rebased pulls and forced pushes.

The method doesn't use `checkout`.
Instead, it uses a different source tree for each branch.

### Warning

If you have commits on your `SOURCE-BRANCH` branch, or any other of your
branches, **_don't use this method on them_**.  You could lose work!
It should be used with new branches created with `urbranch`.

### Preliminaries

Create a github account and fork the urbit repository.

Configure git:

    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com

You'll push to explicitly named branches (thereby overriding this)
but this is a good noob setting.

    git config --global push.default simple

### Create a mirror clone of the github urbit repository

    cd ~
    mkdir URBIT
    cd URBIT

    git clone --mirror https://github.com/urbit/urbit

This is the only time you'll clone from the github urbit repository.
You'll clone from the local repository for all the branches you create.

#### Updating the mirror clone

You'll want to keep your local repository updated.
You'll do this often:

    cd ~/URBIT/urbit.git
    git remote update

The shell script `urbup` listed near the bottom of this file
automates this action.

This document will use `urbup` for brevity.

### Create `master` branch from the local repository

You may want to keep the `master` branch on your
github urbit repository fork updated.

    cd ~/URBIT
    git clone -b master urbit.git urbit-master
    cd urbit-master

You'll want to pull from the local repository with a rebase to
keep your commits at the tip without a merge commit.

    git config --local branch.master.rebase true

Add your github urbit repository fork as a remote:

    git remote add GITHUB-USER https://github.com/GITHUB-USER/urbit

Set it as the default remote to push to:

    git config --local remote.pushdefault GITHUB-USER

Set the remote branch to push to:

    git config --local remote.GITHUB-USER.push +master:master

Note the `+` in the refspec.  It allows pushing to `master` if it's
not in a fast-forward state.  (Essentially, this is forced pushing.)

The shell script `urbranch` listed near the bottom of this file
automates the creation of branches.

#### Updating the `master` branch

    cd ~/URBIT/urbit-master
    urbup
    git pull
    git push

### Create `NAMED-BRANCH` "branch" from the local repository

Note that "branch" is in quotes because you'll clone the `SOURCE-BRANCH` branch.
(`SOURCE-BRANCH` can also be a tag.)

    cd ~/URBIT
    git clone -b SOURCE-BRANCH urbit.git urbit-NAMED-BRANCH
    cd urbit-NAMED-BRANCH

So if you

    git branch -l

you'll see `* SOURCE-BRANCH` (or `* (no branch)` if you cloned from a tag).

You can know which branch you're in by looking at the directory name.

    git config --local branch.SOURCE-BRANCH.rebase true

Note that you specify `SOURCE-BRANCH` instead of `NAMED-BRANCH`.

    git remote add GITHUB-USER https://github.com/GITHUB-USER/urbit
    git config --local remote.pushdefault GITHUB-USER
    git config --local remote.GITHUB-USER.push +SOURCE-BRANCH:NAMED-BRANCH

#### Updating the `NAMED-BRANCH` branch

    cd ~/URBIT/urbit-NAMED-BRANCH
    urbup
    git pull
    git push

The first `git push` will create `NAMED-BRANCH` on github for you.

You will do this regularly to keep this branch up to date.

#### Commit to your branch

    [ edit SOME-FILE ]
    git add SOME-FILE
    git commit -m "SOME COMMIT MESSAGE"
    git push

Your `NAMED-BRANCH` branch will logically be `SOURCE-BRANCH` plus your
commits at the tip.  This will stay true even as you add more of your
own commits while also getting commits from the urbit repository.

#### Compare `NAMED-BRANCH` with `SOURCE-BRANCH`

https://github.com/urbit/urbit/compare/SOURCE-BRANCH...GITHUB-USER:NAMED-BRANCH

The Goal has been achieved.

### Repository update shell script `urbup`

````
#!/bin/sh

# Copyright (c) 2015 John Franklin (jfranklin9000@gmail.com)
# Licensed under the MIT license.

# This is where the everything lives.
URBIT="`echo ~/URBIT`"

# So go there.
if [ -d $URBIT ]
then
	echo "Updating $URBIT/urbit.git"
	cd $URBIT/urbit.git
	git remote update
else
	echo "$URBIT doesn't exist"
	exit
fi
````

### Branch creation shell script `urbranch`

````
#!/bin/sh

# Copyright (c) 2015 John Franklin (jfranklin9000@gmail.com)
# Licensed under the MIT license.

if [ $# -ne 3 ]
then
	echo "usage: urbranch <github-user> <source-branch-or-tag> <named-branch>"
	exit
fi

# Command line args.
usr=$1					# GITHUB-USER
src=$2					# SOURCE-BRANCH (examples: master (branch), v0.4.4 (tag))
dst=$3					# NAMED-BRANCH (example: nock-decrement-opcode)

# We'll clone into this directory.
dir="urbit-$dst"

# This is where the everything lives.
URBIT="`echo ~/URBIT`"

# So go there.
if [ -d $URBIT ]
then
	echo "\nEntering $URBIT"
	cd $URBIT
else
	echo "$URBIT doesn't exist"
	exit
fi

# Create the branch if we can.
git clone -b $src urbit.git $dir
if [ $? -ne 0 ]
then
	exit
fi

# Configure the branch.
cd $dir

# Refer to URBIT-GIT-WORKFLOW.md for explanations of these commands.
git config --local branch.$src.rebase true
git remote add $usr https://github.com/$usr/urbit
git config --local remote.pushdefault $usr
git config --local remote.$usr.push +$src:$dst

echo "\nTo push this branch to GitHub execute these commands:"
echo "cd $URBIT/$dir"
echo "git push"
````

#### Shell script `urbranch` usage

You could have created your branches this way:

    urbranch GITHUB-USER master master
    urbranch GITHUB-USER master NAMED-BRANCH

### Summary

#### Create `ANOTHER-BRANCH`

    urbranch GITHUB-USER master ANOTHER-BRANCH

#### Update `ANOTHER-BRANCH`

    cd ~/URBIT/urbit-ANOTHER-BRANCH
    urbup
    git pull
    git push
