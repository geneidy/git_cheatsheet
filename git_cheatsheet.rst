##########
Git Cheatsheet
##########
 
git
===

tell git who you are
    ``git config --global user.email "you@example.com"``

    ``git config --global user.name "Your Name"``

clone a repository
    ``git clone [repository URL]``


checkout
    ``git checkout [branch]`` switches to another branch

    ``git checkout -b [new-branch]`` creates and switches to a new branch

    ``git checkout -b [new-branch] [existing branch]`` creates and
    switches to a new branch based on an existing one

    ``git checkout -b [new-branch] [remote/branch]`` creates and
    switches to a new branch based on ``remote/branch`` 
    
    ``git checkout [commit sha]`` checks out the state of the repository at a
    particular commit

current status of your local branches 
    ``git status``

show the commit you're on in the current working directory 
    ``git show``

commit
    ``git commit -m "[your commit message]"``
    
add
    ``git add [filename]`` adds a file to the staging areas   

    ``git add -u [filename]`` - the ``-u`` flag will also remove deleted files  
    
remote
    ``git remote add [name] [remote repository URL]`` sets up remote

    ``git remote show`` lists remotes
    
    ``git remote show -v`` lists remotes and their URLs    

branch
    ``git branch``

    ``git branch -a`` to show remote branches too  
    
fetch
    ``git fetch`` gets the latest information about the branches on the default
    remote
    
    ``git fetch [remote]`` gets the latest information about the branches on the
    named remote
    
merge
    ``git merge [branch]`` merges the named branch into the working directory

    ``git merge [remote/branch] -m "[message]"`` merges the branch referred to
    into the working directory - **don't forget to fetch the remote before the
    merge**
    
pull
    ``fetch`` followed by ``merge`` is often safer than ``pull`` - don't assume
    that ``pull`` will do what you expect it to

    ``git pull`` fetches updates from the default remote and merges into the
    working directory

push
    ``git push`` pushes committed changes to the default remote, in branches
    that exist at both ends

    ``git push [remote] [branch]`` pushes the current branch to the named
    ``branch`` on ``remote``
        
log
    ``git log`` will show you a list of commits
    ``git log --oneline`` is useful
    ``git lol`` -> ``git config --global alias.lol "log --graph --decorate --pretty=oneline --all --abbrev-commit"``


Notes
-----

Throughout Git, anything in the form ``remote/branchname`` is a reference, not
a branch.

Tips & Tricks
=============

The "I forgot something in my last commit" Trick
-------------------------------------------------
first: stage the changes you want incorporated in the previous commit
 
use -C to reuse the previous commit message in the HEAD
    ``git commit --amend -C HEAD``
or use -m to make a new message
    ``git commit --amend -m`` 'add some stuff and the other stuff i forgot before'

The "Oh crap I didn't mean to commit yet" Trick
-----------------------------------------------
undo last commit and bring changes back into staging (i.e. reset to the commit one before HEAD)
    ``git reset --soft HEAD^``
    
The "That commit sucked!  Start over!" Trick
--------------------------------------------
undo last commit and destroy those awful changes you made (i.e. reset to the commit one before HEAD)
    ``git reset --hard HEAD^``
    
The "Oh no I should have been working in a branch" Trick
--------------------------------------------------------
takes staged changes and 'stashes' them for later, and reverts to HEAD. 
    ``git stash``
 
creates new branch and switches to it, then takes the stashed changes and stages them in the new branch.   fancy!
    ``git stash branch new-branch-name``

The "OK, which commit broke the build!?" Trick
----------------------------------------------
Made lots of local commits and haven't run any tests...
[unittest runner of choice]
Failures... now unclear where it was broken.

# git bisect to rescue. 
    ``git bisect start`` # to initiate a bisect
    ``git bisect bad``   # to tell bisect that the current rev is the first spot you know was broken.
    ``git bisect good`` <some tag or rev that you knew was working>
    ``git bisect run`` [unittest runner of choice]
# Some runs.
# BLAMO -- git shows you the commit that broke
    ``git bisect reset`` #to exit and put code back to state before git bisect start
# Fix code. Run tests. Commit working code. Make the world a better place.

The "I have merge conflicts, but I know that one version is the correct one" Trick, a.k.a. "Ours vs. Theirs"
------------------------------------------------------------------------------------------------------------
# in master
$ git merge a_branch
CONFLICT (content): Merge conflict in conflict.txt
Automatic merge failed; fix conflicts and then commit.
$ git status -s
UU conflict.txt
 
# we know the version of the file from the branch is the version we want.
$ git checkout --theirs conflict.txt
$ git add conflict.txt
$ git commit
 
# Sometimes during a merge you want to take a file from one side wholesale.
# The following aliases expose the ours and theirs commands which let you
# pick a file(s) from the current branch or the merged branch respectively.
#
# N.b. the function is there as hack to get $@ doing
# what you would expect it to as a shell user.
# Add the below to your .gitconfig for easy ours/theirs aliases. 
#    ours   = "!f() { git checkout --ours $@ && git add $@; }; f"
#    theirs = "!f() { git checkout --theirs $@ && git add $@; }; f"

Split a subdirectory into a new repository/project
--------------------------------------------------
$ git clone ssh://stash/proj/mcplugins.git
$ cd mcplugins
$ git checkout origin/master -b mylib
$ git filter-branch --prune-empty --subdirectory-filter plugins/mylib mylib
$ git push ssh://stash/proj/mylib.git mylib:master
 

Local Branch Cleanup
--------------------
# Delete local branches that have been merged into HEAD
$ git branch --merged | grep -v '\\*\\|master\\|develop' | xargs -n 1 git branch -d
# Delete local branches that have been merged into origin/master
$ git branch --merged origin/master | grep -v '\\*\\|master\\|develop' | xargs -n 1 git branch -d
# Show what local branches haven't been merged to HEAD
$ git branch --no-merged | grep -v '\\*\\|master\\|develop'
