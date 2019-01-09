---
layout: page
title: simple site
tagline: Easy websites with GitHub Pages
description: Minimal tutorial on making a simple website with GitHub Pages
---
# Describes the workflow

Assume that the local repository is correctly set up to point to an upstream copy (origin) and has a local branch called master that tracks the remote master.

    > git remote show origin
  
should return something like this:

    remote origin
    Fetch URL: /Users/peterharrison/tmp/git-local/working.git
    Push  URL: /Users/peterharrison/tmp/git-local/working.git
    HEAD branch: master
    Remote branch:
      master tracked
    Local ref configured for 'git push':
      master pushes to master (up to date)

## local and remote branches

* The upstream repository normally contains a single branch - 'master'
* Content in master is always good-to-go. It is a ready to publish working version of the product. 
* The history of the master branch should be clean and linear. When features are added or changes made, the commits shown in master should follow a clean, linear pattern with a logical progression. Clean up development commits before they are merged into the master branch. Use 'git rebase -i' for this
* Where possible, local feature branches are deleted when a feature is complete. 
* Exceptionally, a feature branch may be pushed upstream temporarily. For example, if development is handed over to another person.
* Pull requests will generate temporary feature branches upstream


## Adding a feature

In the simplest case the developer will just want to modify the latest version in master. Thus the first step is to prepare the local environment

**Make the local environment up to date with upstream origin**

    > git checkout master
    > git pull master   #[1]

At this stage your repo history may look like this (the * at the end of a branch name indicates the currently checked out branch):

    B---C---D master*

**Create a local development branch and get ready to work**

    > git checkout -b feature

This creates a new branch called feature with no commits and chcks it out as the current branch:

              feature*
             /
    B---C---D master

**Optionally tag the start of your work so you can find it easily later**

    > git tag start-feature

Note that the tag should be unique. If you work on several features at the same time, then use a tag name that is related to the actual feature being worked on. Perhaps you could use start-<branch-name>. You are now in a feature branch. 

Commits are made here as and when needed. Commit often. Keep changes granular. For example, if you rename some variables, you might want a commit after each variable is renamed so that you can unpick one from another later if it all goes wrong. After a while, you might consider your feature complete and tested. The local commit history may be untidy and confusing and may not tell the story well. You can tidy it up. Git interactive rebase is the tool to do this. You will get a file containing a list of commits which can be re-ordered, joined and have their messages edited.

After a few commits in the feature brach, your repos may now look like this:

              P---Q---R---S---T feature*
             /
    B---C---D master

IF and ONLY IF you have not modified your master branch, or pushed any changes up to any of the remotes, you can modify the entire feature branch history by doing an interactive rebase on master since that is the place the feature started. If master has changed locally, that will not do what you want. Instead, you have to find the start of the feature branch. If you followed the advice above, it will be tagged. This is a safer approach

**Tidy the feature branch history**

Interactive rebasing lets you squash commits together and edit commit messages. This is handy if you have a series of small commits (as you should) that all add up to a single change. Or perhaps your commit messages are poor. Or maybe you just want to make it look like you worked magic in one clever change.

    > git rebase -i start-feature
 
Quite possibly, you did not tag the start of the feature branch or have some other difficulty locating the place in master where the feature branch starts. There is a somehat arcane git command that will tag the common ancestor of two branches:

    > git tag start-feature $(git merge-base feature master)

For this example, suppose you desided just to squash the last three commits into one, leaving the repo looking like this:

              P---Q---R feature*
             /
    B---C---D master

##Prepare to Share##

After the interactive rebase, you should have a local feature branch history that can be added to the end of the master branch and will form the next published version.

Now you need to make sure that the new feature will work with the current state of master. If you are the only developer and only work on one feature at a time, this will be easy - you already know it works. But, if the master branch has changed for any reason, you need to test your changes. The master branch may have been changed by other developers or you may have changed it in some other operation - perhaps a bugfix, or another feature.

After making sure there are no uncommited changes in the feature branch, you need to switch back to master and bring it up to date. Everything should work without problems because we have a rule that says master is always good and we are just bringing master up to date. 

At this point, the feature branch could be rebased onto master and any issues dealt with there but that would leave your local master in an uncertain state. It is better to switch back to the feature branch and rebase master onto that and then sort out any issues. This way, only the feature branch is potentially broken.
   
**Bring master up to date from the remote**
 
    > git checkout master
    > git pull

**Return to the feature branch and incorporate the new master**

Your repo may now look something like this:

              P---Q---R feature
             /
    B---C---D---E---F---G master*

You want to have the feature tacked onto the end of master

    > git checkout feature
    > git rebase master feature          # rebase feature onto master

And the repo should look like this:

                          P'---Q'---R' feature*
                         /
    B---C---D---E---F---G master

Now you have an up to date copy of master along with your changes from your feature branch. If master has not changed, this will cause no problems and nothing will be committed so you can just carry on.

If master has changed while you were working on the feature then your changes may have caused conflicts. These will need to be resolved and changes made until everything is working again.

It is possible that this is an involved process and master changes some more in which case you might have to go around a couple of times. Whoever is in charge of maintaining the upstream repository should use a mechanism to make sure that this is unlikely.

For a solo developer, these conflicts are less likely unless you are working on more than one feature. You may, for example, have to temporarily pause work on the feature and do a bug fix on the master branch. Commits E, F and G above may represent that change

Finally, you have a working feature on an up to date copy of master but all the commits are only in your feature branch so the master branch stil knows nothing about them. You should also have an up to date master. 

Now you need to get your changes into the local master and push them upstream. (Note that managed repositories will use a pull request to incorporate changes. That is not dealt with here).



**Incorporate the feature into master and push it up.**

    > git checkout master
    > git rebase feature
    > git merge feature
    
The git merge command will do a fast-forward and just bring the head of master up to the most recent commit. Now you should have a repo looking like this:
      
                                        feature
                                       /
    B---C---D---E---F---G---P'---Q'---R' master*

If it is important to track this version/revision/build, you can add a tag to the master branch.
    
    > git tag V18.09.15.a

The feature is complete so we can delete the feature branch. Remember that branches are just names for a particular commit. No commits are harmed in the deleting of a branch although it is possible that the commits become hard to find.

**Tidy up**

    > git tag                       # list the tags - just to check
    > git tag -d start-feature      # only if you tagged the feature start
    > git branch -d feature         # delete the feature branch 

Now the repo is:

                                        V18.09.15.a
                                       /
    B---C---D---E---F---G---P'---Q'---R' master*

**Push the changes.**

At last, you are ready to push the changes upstream so that specific versions can be pulled as needed at a future date. Push master because that is always the current known-good system. Also push the version tag to that you can always grab that specific version of the system. That may be necessary, for example, if you want to do a bug fix for an older releas of the system that is already out in a product.

    > git push origin master
    > git push origin V18.09.15.a


When you are sure all is done, remember to tidy up. You can delete the local feature branch and temporary tags to save clutter. The tags and branches can be listed out so that you can find them easily.

And you are done. The feature is complete and master is up to date.

**[1] alternative to pull, described by Miguel Sánchez de León Peque.** 

Here, 'origin' could also be bitbucket or whatever you called your remote when setting it up

Fetch the latest changes, but no need to update local master

    > git fetch origin master 
  
Rebase directly against origin/master, not my local master which may be outdated

    > git rebase -i origin/master  

Force-push to origin after rebase, to update the pull-request/merge-request and trigger possible CI pipelines

    > git push -f origin feature  
  
If everything went well with tests, reviews, etc., push to upstream (or use GitHub/Gitlab/whatever web interface instead to merge). If the upstream/master changed, it will complain and I will have to repeat the process

    > git push origin feature:master  
  
  
