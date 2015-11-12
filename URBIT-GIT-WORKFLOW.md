# ~ripnul-rilnyx urbit git workflow

Copyright (c) 2015 John Franklin (jfranklin9000@gmail.com)

Licensed under the MIT license.

## The Goal

Create long lived branches on your github urbit repository fork with
your commits at the tips of the branches.

You want to create
`https://github.com/GITHUB-USER/urbit/tree/URBIT-BRANCH`
and keep it up to date with
`https://github.com/urbit/urbit/tree/test`
with your commits at the tip of the branch.

Example:

    GITHUB-USER = jfranklin9000
    URBIT-BRANCH = nock-decrement-opcode

### Notes

This document uses hardwired paths.

This is where everything lives:

    ~/URBIT

This is where branch `URBIT-BRANCH` lives:

    ~/URBIT/urbit-URBIT-BRANCH

You can adjust these as you see fit.

This document assumes all pulls don't require manual conflict resolution.
Standard merge techniques apply if that's not the case.

The method of achieving The Goal uses rebased pulls and forced pushes.

The method doesn't use `checkout`.
Instead, it uses a different source tree for each branch.

### Warning

If you have commits on your `test` branch, or any other of your branches, **_don't use this method on them_**.  You could lose work!
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

    git clone --mirror git://github.com/urbit/urbit.git

This is the only time you'll clone from the github urbit repository.
You'll clone from the local repository for all the branches you create.

#### Updating the mirror clone

You'll want to keep your local repository updated.
You'll do this often:

    cd ~/URBIT/urbit.git
    git remote update

Here's a command you can add to your `.bash_login` or `.bashrc`:

    alias urbup='pushd ~/URBIT/urbit.git; git remote update; popd'

This document will use `urbup` for brevity.

### Create `master` branch from the local repository

If you want to build and run urbit with a planet it's suggested
that you do it from the `master` branch.
However, you won't be committing to this branch.

    cd ~/URBIT
    git clone -b master urbit.git urbit-master
    cd urbit-master

Although you won't be committing to this branch you (may) want to keep
the `master` branch of your github urbit fork up to date.

You'll want to pull from the local repository with a rebase to
keep your commits at the tip without a merge commit.
(Right, you're not committing to this branch, but in case you ever do...)

    git config --local branch.master.rebase true

Add your github urbit fork as a remote:

    git remote add GITHUB-USER https://github.com/GITHUB-USER/urbit

Set it as the default remote to push to:

    git config --local remote.pushdefault GITHUB-USER

Set the remote branch to push to:

    git config --local remote.GITHUB-USER.push +master:master

Note the `+` in the refspec.  It allows pushing to `master` if it's
not in a fast-forward state.  (Essentially, this is forced pushing.)

There's a shell script listed near the bottom of this file to
automate the creation of branches.

#### Updating the `master` branch

    cd ~/URBIT/urbit-master
    urbup
    git pull
    git push

### Create `test` branch from the local repository

You generally won't commit to this branch except, perhaps, for small
pull requests that can be quickly evaluated and pulled.  In any case,
you'll want to follow this important development branch.

    cd ~/URBIT
    git clone -b test urbit.git urbit-test
    cd urbit-test

    git config --local branch.test.rebase true
    git remote add GITHUB-USER https://github.com/GITHUB-USER/urbit
    git config --local remote.pushdefault GITHUB-USER
    git config --local remote.GITHUB-USER.push +test:test

#### Updating the `test` branch

    cd ~/URBIT/urbit-test
    urbup
    git pull
    git push

### Create `URBIT-BRANCH` "branch" from the local repository

Now we get to the point and address The Goal.
Note that "branch" is in quotes because you'll clone the `test` branch.

    cd ~/URBIT
    git clone -b test urbit.git urbit-URBIT-BRANCH
    cd urbit-URBIT-BRANCH

So if you

    git branch -l

you'll see `* test`.

You can know which branch you're in by looking at the directory name.

    git config --local branch.test.rebase true

Note that you specify `test` instead of `URBIT-BRANCH`.

    git remote add GITHUB-USER https://github.com/GITHUB-USER/urbit
    git config --local remote.pushdefault GITHUB-USER
    git config --local remote.GITHUB-USER.push +test:URBIT-BRANCH

#### Updating the `URBIT-BRANCH` branch

    cd ~/URBIT/urbit-URBIT-BRANCH
    urbup
    git pull
    git push

The first `git push` will create `URBIT-BRANCH` on github for you.

You will do this regularly to keep this branch up to date.

#### Commit to your branch

    [ edit SOME-FILE ]
    git add SOME-FILE
    git commit -m "SOME COMMIT MESSAGE"
    git push

Your `URBIT-BRANCH` branch will logically be `test` plus your commits
at the tip.  This will stay true even as you add more of your own
commits while also getting commits from the urbit repository.

#### Compare `URBIT-BRANCH` with `test`

https://github.com/urbit/urbit/compare/test...GITHUB-USER:URBIT-BRANCH

The Goal has been achieved.

### Branch creation shell script `urbranch`

````
#!/bin/sh

# Copyright (c) 2015 John Franklin (jfranklin9000@gmail.com)
# Licensed under the MIT license.

if [ $# -ne 2 ]
then
	echo "usage: urbranch <github-user> <urbit-branch>"
	exit
fi

# Command line args.
ghuser=$1					# GITHUB-USER
branch=$2					# URBIT-BRANCH

# We'll clone into this directory.
branchdir="urbit-$branch"

# Source branch to clone from.
# Only 'master' and 'test' exist.
# If $branch is 'master' $src will be 'master',
# else $src will be 'test'.
if [ $branch = "master" ]
then
	src="master"
else
	src="test"
fi

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
git clone -b $src urbit.git $branchdir
if [ $? -ne 0 ]
then
	exit
fi

# Configure the branch.
cd $branchdir

# Refer to URBIT-GIT-WORKFLOW.md for explanations of these commands.
git config --local branch.$src.rebase true
git remote add $ghuser https://github.com/$ghuser/urbit
git config --local remote.pushdefault $ghuser
git config --local remote.$ghuser.push +$src:$branch

echo "\nTo push this branch to GitHub execute these commands:"
echo "cd $URBIT/$branchdir"
echo "git push"
````

#### Shell script `urbranch` usage

You could have created your branches this way:

    urbranch GITHUB-USER master
    urbranch GITHUB-USER test
    urbranch GITHUB-USER URBIT-BRANCH

The script will helpfully `cd` to `~/URBIT` for you.

### Summary

#### Create `ANOTHER-BRANCH`

    urbranch GITHUB-USER ANOTHER-BRANCH

#### Update `ANOTHER-BRANCH`

    cd ~/URBIT/urbit-ANOTHER-BRANCH
    urbup
    git pull
    git push
