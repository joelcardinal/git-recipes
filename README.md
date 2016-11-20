# Git Workflow Recipes

This document assumes the following basic branching strategy:
- dev : branch for sprint work
- XXX : branch representing a release candidate, the XXX would be a numerical identifier of the build
- master : branch that will be kept at same state as Production. The XXX branch will be merged into master after a Production release.

It's best to first read this from top to bottom, as many of the commands are used for multiple recipes they are typically only explained once when first mentioned.

The goal of this document is meant to provide very basic "how to get stuff done", utilizing the most basic git commands in the terminal.  It is not meant to fully explain git nor the intricacies of each command, I highly encourage you to take a deep dive into learning git.

For more information about additional commands and online resources checkout my git cheat sheet: https://github.com/joelcardinal/git-cheat-sheet


### Create new bug ticket issue branch

First thing you should always do is check what branch you are on, and if you have any modified files not yet committed.

```git status```

For this example, we'll assume you are on some random branch and no modified files.  We'll first switch to master branch.

```git checkout master```

Make git aware of any changes on the remote repo.

```git fetch origin```

NOTE: It's important to remember the command above only makes git aware of changes, but it has not brought any local branches up-to-date.

For this example let's assume we want to create a new branch to work on a bug ticket.

```git checkout -b 19872_Fix-Header-Nav origin/dev```

Above we are telling git that we want to switch to a new branch called 19872_Fix-Header-Nav and we want the commit history to be based off of the remote dev branch.

If we had not performed the git fetch origin, our issue branch would have been based off stale history of dev and not have the latest changes that are on dev.

Now you are free to do your work, when you are ready to commit see section below.


### Commit work on bug ticket issue branch

This section assumes you are on your bug ticket issue branch and are ready to commit.  First, always verify you are on the correct branch and only the expected files have been created or modified.

```git status```

Using the command above, git will tell you what branch you are on and what files have been created, changed, or deleted.

NOTE: git does not track empty directories, if you need to add an empty directory for some reason, you will need to add a temporary file.

Let's assume that only the expected files are listed by git, we need to tell git that we are ready to commit the files, this is called staging.

```git add -A```

The -A option tells git that all the listed modified files should be staged, this can be dangerous if you don't use git status first and verify, because you might end up staging and then committing a file you didn't want committed.

Also note that even deleted files need to be added, it sounds counter-intuitive, but again, we're just telling git that we plan on committing these changes.

Double-check that the expected modified files are now staged.

```git status```

By now you should realize that git status is your best friend, and you should use it as much as possible, it helps you double-check the state of git.

Let's assume only expected files are staged, we need to now commit the changes.

```git commit -m "19872 Fixed Header Nav"```

Above we used the -m option which allows us to add a short commit description inline.  If we left off the -m option git would open your default terminal editor where it would expect you to write your commit message and save the message.  There is a global git config you can use to set your preferred editor.

Now we need to push up the local bug ticket issue branch to the remote host so that we can issue a pull request

```git push origin 19872_Fix-Header-Nav```

Above it is important that you use your correct branch name, for this example we used 19872_Fix-Header-Nav.

If your team requires submitting pull requests, you'll likely need to use your hosting environment (GitHub, BitBucket...etc.) to initiate the pull request.  This typically involves navigating to your issue branch under "Branches", then clicking a button to create a pull request, and correctly selecting the branch you want your issue branch merged into, in this example we want our issue branch merged into dev branch.


### My pull request was disapproved because it couldn't be merged in

Using our example above, let's assume we have bug ticket issue branch 19872_Fix-Header-Nav.  We've already worked on it, committed it, pushed it up and submitted a pull request.  But the reviewer refuses to merge it in because there is a conflict with the branch it is to be merged into, let's assume that is the dev branch.  We need to bring our issue branch up-to-date with dev.

First thing you should always do is check what branch you are on, and if you have any modified files not yet committed.

```git status```

For this example, we'll assume you are on your issue branch and no modified files.  We'll checkout master and update git's reference to remote branches.

```
git checkout master
git fetch origin
```

Now that git knows the lastest changes to the dev branch let's bring those changes into our issue branch.

```git merge origin/dev```

Notice the slash used, we are telling git we want to merge in the dev history found on remote, not any local copy of the dev branch we may have (which you likely don't have).

There are two outcomes when merging, either it was successful or it was not and git says there is a conflict.  Let's assume there was no conflict, the section below discusses what to do when you get a merge conflict.

Sometimes if a merge is successful git will want you to add a message, and after issuing the merge command it will immediately open a terminal editor.  A default message will already be populated, you can use it or add your own.  The reason git makes you add a message is that the merge is actually creating a commit, and all commits need a message.  Simply save your message.

If the merge is a success the only thing left to do is push the issue branch back up to remote.

```git push origin 19872_Fix-Header-Nav```

You may need to add a comment to your pull request or resubmit your pull request so the reviewer knows you've updated your issue branch.

IMPORTANT: You may have read or heard about updating a branch using git pull instead of using fetch origin and merge; read section "How to work with someone else on a bug" to learn the dangers of git pull.

### How to handle merge conflicts

This section assumes you've just issued command git merge and git states a merge conflict has occured.  Part of the messaging will state the files where a conflict was found, but if you need to see again which files have the conflict once again using status is your friend.

```git status```

You have two choices, you can abort the merge which takes your branch back to the state before you issued the merge command, or you can resolve the conflicts.  Let's look at both options, here is the command to abort the merge.

```git merge --abort```

Now we'll explore resolving the conflict.  Just like other versioning methods, git will wrap the problem lines of code in each file it finds conflict.  It will look something like this in the code file when you open in and editor.

```
<<<<<<< HEAD
var test = 'Hello World!';
=======
var test = 'Hello Cruel World?';
>>>>>>> origin/dev
```

Let's assume you were trying to merge remote branch dev into your local issue branch 19872_Fix-Header-Nav.  The top "<<<<<<< HEAD" section represents the code lines from 19872_Fix-Header-Nav, and the latter ">>>>>>> origin/dev" section is what it found on the remote dev branch that conflicts.

IMPORTANT: Sometimes the wrapping conflict "<<<<<<<" identifiers are hard to spot, in the wrong section, and there may be mutiple sections that need fixing, review the entire document carefully.

You'll need to manually remove all unnecessary code and the conflict "<<<<<<<" identifiers.  If our intention is to keep the work we did in 19872_Fix-Header-Nav and discard the code from the dev branch the correction would look like this.

```
var test = 'Hello World!';
```

After fixing all conflicts in all files be sure to save the files.  Now that the files have been corrected let's double-check we corrected them all.

```git status```

All the files you corrected should now be in the modified status, there should be no files with conflict, if there is, resolve them.

You'll notice git says that the it found Unmerged paths, you haven't yet resolved the merge conflict.  All this means is that you need to stage and commit these changes before git will see this merge conflict as resolved.

```
git add -A
git status
git commit -m "19872 Resolved merging in dev into 19872_Fix-Header-Nav"
```

Above in the first line we added all modified files, on the second we verified only the files we wanted staged are there, and the last we commit the fixes.


### How to work with someone else on a bug

This section assumes that someone has an issue branch, has pushed up the issue branch to remote and you need to pull it down, work on it, and push it back up.  We are assuming you do not yet have a local copy of the branch.  Let's assume the remote branch is "19872_Fix-Header-Nav".

```
git status
git checkout master
git fetch origin
```

We've covered the necessity of the commands above in other sections.  Now we are going to create a local copy of remote branch 19872_Fix-Header-Nav, and we are going to switch to it as our active branch.

```git checkout -b 19872_Fix-Header-Nav origin/19872_Fix-Header-Nav```

Now you can do your work.

When you are ready to commit you can follow the same commands we've covered in other sections above.

```
git status
git add -A
git commit -m "19872 tweaked title alignment"
git push origin 19872_Fix-Header-Nav
```

When you attempt to push a local branch up to remote, where the same branch already exists there are two possible outcomes, it will be a success, or the push to remote will fail.  When it fails, this typically means someone has pushed up a commit that you don't yet have, and git wants you to bring your local branch up-to-date before pushing.

There are two ways to update your local branch, one is safer than that other.

1. Here are going to use a short-cut command which does two actions, fetches the changes from remote branch, and merges them in.

```git pull```

The problem with this method is how git knows what remote branch to merge in.  This will only work correctly if you create the local branch with the reference to origin as we did above ```git checkout -b 19872_Fix-Header-Nav origin/19872_Fix-Header-Nav```.

This portion ```origin/19872_Fix-Header-Nav``` makes git aware of what remote branch gets tracked against the local branch you created.

The danger is that, if you forget to state the remote branch when you use ```git pull``` it may not use the correct branch.

2. The longer but safer way.

```
git status
git checkout master
git fetch origin
git checkout 19872_Fix-Header-Nav
git merge origin/19872_Fix-Header-Nav
```

Moving on, we'll assume there was no merge conflict, if there is see merge conflict section.

Now try pushing again.

```git push origin 19872_Fix-Header-Nav```

Let's now assume that you go to lunch and when you come back your teammate says that they've made another change and want you to do some additional work.  Let's assume that you have not deleted your local 19872_Fix-Header-Nav branch, it exists locally, but let's also assume your active branch is some random branch.

```
git status
git checkout 19872_Fix-Header-Nav
git pull
```

We'll assume you didn't see any modified files when you performed the git status and git pull didn't lead to a merge conflict.  We've covered the commands above already, so it should make sense what we are doing.  You are now ready to do your additional work.


### How to fix QA/UAT issue on branch XXX

This workflow is nearly the same as "Create new bug ticket issue branch", with some minor changes, so I'll just point out the differences.

```
git status
git checkout master
git fetch origin
git checkout -b 19872_QA-Bug origin/XXX
git status
git add -A
git commit -m "19872 Fixed QA bug"
git push origin 19872_QA-Bug
```

Notice above when we create and switch to a new bug we point the remote tracking branch to the remote XXX branch, doing so allows use to use git pull in case the push to origin fails due to new commits on remote which we discussed in another section above.

With your issue branch pushed up, you are ready to issue a pull request using your remote repo host's interface.  When you perform the pull request you'll want to set the branch you wish to have your issue branch merged into as XXX.


### How to work on a hotfix

In the branching strategy we described at the top of the page there should typically be only one XXX branch which is the release candidate, which is undergoing QA/UAT.

This section assumes there is a particualar branching strategy for a hotfix, where a XXX branch is created from master and hotfix work initiates a pull requrest on this hotfix XXX branch.

Let's first clarify how branches are maintained.  Typically, there will be two branches, dev and master.  Sprint issue branches are branched off dev and there subsequent pull requests for these issue branches will be against dev, as we've already explored in sections above.

When a sprint is ready for QA/UAT and new XXX branch is created off of dev branch.

After a release the XXX branch is merged into master then deleted.

Let's run through a scenario.  First we start with dev and master.

```
dev
master
```

A sprint is over and we branch off dev to create our XXX release candidate branch, we'll call it 250.

```
dev
master
250
```

Now let's say 250 is released, it then gets merged into master.

```
dev
master
```

Now let's say a new sprint is ready to be branched from dev, we'll call it 251.

```
dev
master
251
```

Now assume there is a bad bug in Production and we need a hotfix, we'll need to create the 250 branch again that the developer can issue pull requests against.

```
dev
master
250
251
```

Let's assume the developer's pull request was accepted and 250 was released again, subsequently 250 will get merged into master then deleted.

```
dev
master
251
```

That was an overview of the branching strategy, let's go over what a developer needs to do.

Using our example above, let's go back in time to the point where sprint 251 is ready for QA/UAT.

```
dev
master
251
```

A Production bug is found and a hotfix for 250 is required.  We'll assume there is only one hotfix issue and branch 250 does not exist so we need to create it, then create our hotfix issue branch off of it.  Since the 250 release build is in master we'll branch off master to create the 250 branch. 

```
git status
git checkout master
git fetch origin
git checkout -b 250 origin/master
```

The last line above creates the 250 branch off of remote master, but this 250 branch only exists locally, we still need to push it up to remote.

```git push origin 250```

The 250 branch exists locally and remotely, but we now need to create the hotfix issue branch that we will do our work in.  Note below we track against remote 250.

```git checkout -b 19872_Hotfix-Bug origin/250```

We are ready to do our hotfix work in 19872_Hotfix-Bug.  When we are done, we commit and push up the hotfix issue branch.

```
git status
git add -A
git commit -m "19872 Fixed hotfix bug"
git push origin 19872_Hotfix-Bug
```

You can now issue a pull request for branch 19872_Hotfix-Bug against hotfix branch 250.
