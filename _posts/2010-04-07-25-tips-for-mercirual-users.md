---
layout: post
title: "25 Tips for Mercirual Users"
category: VersionControl
tags: [dvcs,cvs,hg,Mercurial]
---
{% include JB/setup %}

I recently read an interesting article by Andy Jeffries entitled [25 Tips for Intermediate Git Users](http://andyjeffries.co.uk/articles/25-tips-for-intermediate-git-users) (linked to via [proggit](http://www.reddit.com/r/programming/)) .  It had lots of useful information condensed into bite-sized task-oriented chunks.

I've been using [Mercurial](http://mercurial.selenic.com/) for a while now, so I thought I would write a similar set of tips by translating from `git` to the equivalent `hg` commands.  Thanks to Andy for blessing this translation work.  There may well be some mistakes herein - please leave a comment if you have any improvements or fixes to suggest.

# Basic Tips

## 1. First Steps After Install

Once you have [installed Mercurial](http://www.selenic.com/mercurial/download/), you should create the `.hgrc` file in your home directory which contains your default user configuration.  This is where you can customize options and load additional [extensions](http://www.selenic.com/mercurial/wiki/Extensions).  The absolute minimum is to specify your name and email address, which is recorded in every commit you make:

    [ui]
    username = Joe Coder <joe@example.com>

To enable extensions, add a section called `[extensions]` and list the names.  I would recommend the following extensions, which are incredibly useful.  Just add another section to your config file:

    [extensions]
    graphlog=
    pager=
    color=
    fetch=
    record=
    bookmarks=

To configure an extension, add a section with its name, and the options within.  For example, to configure the `pager` extension, you should configure it to not interfere with certain interactive commands.  Something like this:

    [pager]
    pager = LESS='FSRX' less
    ignore = version, help, update, serve, record

For complete information about this file, see:

    $ hg help config

## 2. Mercurial is tree-based

Mercurial stores its repository in the "revlog" format.  This is a tree of commits and their metadata, with each commit being uniquely identified by its the SHA-1 hash.

<img src="http://antonym.org/hg25/MercurialRevisionTreeExample.png" width="703" height="221"  />

When you see a changeset in this format:

    4225:cf1a9b1f9bee

this is showing the simple revision number, `4225`, and the revision hash, `cf1a9b1f9bee`.  Either can be used to refer to a revision, and usually the first four characters of the hash are sufficient to uniquely identify a revision.

The simple revision number (akin to the sequentially numbered revisions in `svn`) is unique to a repository, but not necessarily across repository clones.

Tags are simply labels associated with a revision.  Named branches are stored in the changeset metadata, while bookmarks are much like local tags that move with each commit.

There is always one named branch in existence: `default`.  This is normally where mainline development occurs, and is equivalent to `trunk` in Subversion, or `master` in git.

## 3. Two parents - of course!

The `hg log` command lets you view the details of your commit history.  Normally each commit has one parent, which is the path up the tree.  But when you perform a `merge` operation, you are combining two branches of the tree together.  This is known as a "merge commit", and the changeset is recorded as having two parents, referring to the two branches that were merged.

For example, here is a commit from one of my repos showing r32 merging with r30, to produce r33:

    .
    .
    |
    |
    o    changeset:   33:9710bef2500e
    |\   parent:      30:ad3a51b8aa18
    | |  parent:      32:a3f98e81c19c
    | |  user:        Gavin Baker <gavinb@...>
    | |  date:        Mon Nov 09 07:47:15 2009 +1100
    | |  summary:     Merged udp log work from deadlock branch
    | |
    | o  changeset:   32:a3f98e81c19c
    | |  branch:      deadlock_gb
    | |  user:        Gavin Baker <gavinb@...>
    | |  date:        Mon Nov 09 07:43:51 2009 +1100
    | |  summary:     Started adding udp logging
    | |
    . .
    . .


## 4. Merge conflicts

Mercurial can be configured to use an external tool to process three-way merges.  For example, to use the popular `kdiff3` tool, add the following section to your `.hgrc`:

    [merge-tools]
    # Override stock tool location
    kdiff3.executable = ~/bin/kdiff3
    # Specify command line
    kdiff3.args = $base $local $other -o $output
    # Give higher priority
    kdiff3.priority = 1

If you don't have an external merging tool, mercurial will save the left and right versions of the files, and place conflict markers ("<<<<", "---", ">>>>") in the working copy.  It is up to you to then edit the file, resolve the conflict, and then resolve or commit your changes.  To mark a conflicted file as resolved:

    $ hg resolve -m main.c

To check the status of files involved in the merge:

    $ hg resolve -l

# Servers, Branching and Tagging

## 5. Remote servers

In Mercurial, every repository is self-contained and includes the full history of the project.  When you `clone` a repository, the URI of the original is stored in the repository's own `hgrc` file (stored in the `.hg` directory at the root of the repository).  In the section labelled `paths`, an entry is created called `default`, which points to the original repo.  This way, you do not have to specify a URI for certain commands that involve remote repositories.

To see which changes have been made in the default remote repository since you last updated:

    hg incoming

To see which changes you have locally that have not yet been pushed to the default remote repository:

    hg outgoing

You can always add more entries to the remote server list by editing the `.hg/hgrc` file.  These are simply aliases for the full URI, which you can use along with commands that need a repository path.  For example, given the following entries in the `.hg/hgrc` of a given repo:

    [paths]
    goog = http://code.google.com/p/golang/
    exp = https://bitbucket.org/gavinb/golang-exp/

the following sets of commands are equivalent:

    $ hg incoming http://code.google.com/p/golang/
    $ hg incoming goog
    
    $ hg outgoing https://bitbucket.org/gavinb/golang-exp/
    $ hg outgoing exp

Open questions:

 - Is there an extension equivalent to `git remote` for managing paths?
 - How do you compare just a local and a remote branch (not the full set of changesets)?

## 6. Tagging

In Mercurial, there are two types of tags: regular tags and local tags.  Regular tags are stored in the `.hgtags` file in the root of the repository, and are versioned just like any other file.  These tags are also therefore propagated to any remote servers.  You can list all the current tags with:

    $ hg tags

To add a regular tag to the current revision, you simply give it a name:

    $ hg tag version_1_3

To associate a tag with a specific revision, you use the `-r` option:

    $ hg tag -r 4a93 version_1_2_1

<img alt="MercurialTaggingExample.png" src="http://antonym.org/hg25/MercurialTaggingExample.png" width="702" height="248" />

Once you have created a tag, you can use its name virtually anywhere you would normally specify a revision.  So after the above two operations, `hg tags` would show:

    tip                 152:01ae0a9083c3
    version_1_3          79;08a49fb329a1
    version_1_2_1        22:4a93c0e1aa0b

A local tag works in almost the same way, except local tags do not get copied between repositories when running a `push` or `pull`.  You create a local tag by specifying the `-l` flag:

    $ hg tag -l experimental

The `bookmarks` extension also provides a form of local tags, which move with the latest commit.  (Bookmarks are very much like git's cheap local branches, and a topic worth its own article.)

## 7. Creating Branches

Creating a named branch in Mercurial is a matter of simply specifying a branch name, which will be saved in the metadata of the next commit:

    $ hg branch bug_fix_5948

To switch between branches, use the `update` command with the branch name:

    $ hg update gui_changes_experimental

To display all the current branches, simply issue:

    $ hg branches

The following example shows a feature branch for the GUI, and another for the database layer.  These branches are periodically merged with default, where releases are made:

<img alt="MercurialBranchingExample.png" src="http://antonym.org/hg25/MercurialBranchingExample.png" width="904" height="291" />

There are other types of branches too.  The simplest is an "anonymous" branch, which you can create simply by updating to an earlier revision, making some changes, and committing a new child revision.  This will result in a commit with two children - ie. a branch.  Steve Losh has written [an excellent article on branching](http://stevelosh.com/blog/2009/08/a-guide-to-branching-in-mercurial/) which explains this in more detail.

## 8. Merging Branches

Once the work on your branch is complete, you will want to merge it back to the mainline of development (which is usually `default`).  To close the branch before you merge, use:

    # on branch gui_changes_experimental
    $ hg commit --close-branch -m "Experimental branch was a success"

Then you switch back to the target branch into which you want to merge your changes.

    $ hg update default

And then specify the name of the branch to merge, like this:

    $ hg merge gui_changes_experimental

Mercurial will then attempt to automatically merge all the changes, and notify you if there are any conflicts that require manual resolution.  At this point it can also launch a custom merging utility. (See tip #4 above.)

If you would like to see which changesets would be merged from a branch, you can preview the merge using:

    $ hg merge -P gui_changes_experimental

You can see a list of open branches using:

    $ hg branches

And you can see which changes have yet to be merged by doing a diff with the branch name:

    # on branch default
    hg diff gui_changes_experimental

## 9. Remote branches

By default, when you `push` your changes to a remote server, you push all commits made since the last update.  To push only a single branch, you can specify its name (or a given revision):

    $ hg push -r gui_changes_experimental

This will push all the changesets starting from the head of the branch, and all its ancestors that do not appear in the remote repository.

As far as I know, there is no direct equivalent to git's tracking branches in Mercurial.

# Storing Content in stashes, etc

## 10. Stashing

If you are in the middle of a set of changes, and you need to perform an operation on the repository without first committing all of your local changes, you can "stash" them away and set them aside using the [shelve](http://mercurial.selenic.com/wiki/ShelveExtension) extension.  This puts your current set of changes into a temporary storage area, from which you can restore them later.

To save your changes, run:

    $ hg shelve

You can interactively specify which changes to shelve at the granularity level of individual hunks, or simply stash entire files.  To bring your changes back into the working directory, simply use:

    $ hg unshelve

## 11. Adding Interactively

Ideally every commit should be a discrete patch that applies one cohesive change.  But for various real-world reasons, you often find yourself with a working directory that contains modified files containing multiple unrelated changes.  Sometimes it is individual files, sometimes even within the one file.  It would be great to be able to commit a subset of your patches, and choose individual hunks or files to commit.  The `record` extension does exactly that.  Running:

    $ hg record

interactively shows you each file, and lets you inspect each hunk that has changed, one by one.  You can accept individual hunks to commit, accept or ignore an entire file, and so on.  Any changes you do not commit will remain in your working copy to deal with later.  This is a great way to keep your commits clean and separate.

## 12. Handling Large Files

(There is no direct equivalent to the original list's item #12, "Storing/Retrieving from the Filesystem".)

Some projects (games are an excellent example) feature large binary resources, which do not lend themselves well to management under a VCS.  Examples include images, movies, sound files, level data, and so on.  You don't want to keep multiple versions of these files around if you can help it, and performance can suffer for files over about 10MB.

The [`bigfile` extension](http://mercurial.selenic.com/wiki/BigfilesExtension) aims to solve this problem by storing large binary files (Binary Large Objects, or BLOBs) in a separate location, while still allowing you to manage the files using Mercurial.  A sample session looks like this:

*** TODO Insert bigfile example

# Logging and What Changed

## 13. Viewing a Log

To review the details of your commit history, you use the `log` command.  A brief one-line summary is displayed with:

    $ hg log

To see the logs for a specific revision, range, tag or branch, use the `-r` switch:

    $ hg log -r9584
    $ hg log -r release_1.3

To show all the details of the metadata, and the full commit message, use the verbose `-v` switch.

If you want to view the patch itself, use:

    $ hg log -p

If you enabled the `glog` extension (distributed with Mercurial), you can see a graphical representation (in ASCII art) of the commit tree, showing branches and merges using:

    $ hg glog

## 14. Searching the Log

To display only those changesets which have been committed by a given author, use the following command:

    $ hg log -u joe

You can search for a username, real name or in fact any substring in the 'author' field.

To search for a keyword that appears in the commit message, use:

    $ hg log -k regression

To search for commits that changed certain filenames, you can include or exclude patterns:

    $ hg log -I "**/Makefile"
    $ hg log -X "*.xml"

To view all commits made since a given date:

    $ hg log -d ">2009-11-1"

There are many more date options - see `hg help dates` for the full list.

The `log` command can also display the log output using a user-defined template, which can be very useful for scripting and custom reporting.  This example can be used to derive summary statistics of changed files by user:

    $ hg log --template "{date}|{user}|{diffstat}" -d ">2009-11-20"

See `hg help templating` for the complete syntax.

## 15. Selecting Revisions

There are many ways of specifying a revision when using commands such as `log`, `merge`, and so on.

    $ hg log -r 4281               # By short revision number
    $ hg log -r d1b75410b793       # By full revision number (SHA-1 hash)
    $ hg log -r feature_213        # By branch name (refers to head of branch)
    $ hg log -r release_1_31       # By tag name
    $ hg log -r refactor_engine    # By bookmark name

If you install the [`parentrevspec` extension](http://mercurial.selenic.com/wiki/ParentrevspecExtension), you can also use git-style relative revisions, such as `foo^^` for the second parent of revision `foo`.

## 16. Selecting a range

To specify a range of revisions to a command taking a `-r` revision range, use the syntax `[BEGIN]:[END]`.  If `BEGIN` is not specified, it defaults to revision 0, and if `END` is not specified, the `tip` revision is used.  A single `:` by itself refers to all history.  You can even specify the revisions in reverse order, if `END` is less than `BEGIN`.  Again, a revision can be specified using any style as per tip #15 above.

# Rewinding Time and Fixing Mistakes

## 17. Resetting changes

If you have modified a file in your working directory, but you wish to discard the changes and not commit them, you can use:

    $ hg revert parser.c

If you just committed a change that you didn't intend, you can back out that change.  (Note that this will only work if the commit was the very last transaction you applied.)  You run:

    $ hg rollback

The working copy will be back to where it was before the commit, and the commit will be deleted from the repository.

If you want to delete an entire branch, you can do so (provided you have the `mq` extension enabled) using:

    $ hg strip -r 432

This will remove revision 432 *and all its descendants* from the repository.  So it is just like pruning a branch from a tree.  Note that this is a very destructive operation!  For your safety, Mercurial will store the stripped changesets into a bundle, so you can reapply the bundle if you made a mistake.

The preferred way of undoing the effect of an older commit is to use:

    $ hg backout -r 9684

This will create a new changeset that reverses the effect of the specified revision.  It may create a new head (depending on the working directory's parent), in which case you can merge the new head into default.

## 18. Committing to the wrong Branch

If you've made a bunch of changes, then committed them to the wrong branch in multiple versions, all is not lost.  While `hg rollback` will undo the last transaction (ie. commit), it cannot reverse more.

If you want to undo the changes in one changeset, use:

    $ hg backout

which will create a new changeset that reverses the effect of the specified revision.  This will normally create a new head, so you will need to merge this into `tip`.

If you have committed a series of changesets, you could also run `hg strip` from the errant revision to remove it from the tree.  Then switch to the correct branch, take the bundle file that `strip` saves for you, and apply the bundle.

## 19. Interactive Rebasing

The [`histedit` extension](http://mercurial.selenic.com/wiki/HisteditExtension) provides similar functionality to git's powerful `rebase --interactive` command.  It allows you to take a series of changesets, and commit, fold into a previous commit, edit or drop each revision.  It will then reorder and reapply the changesets in the specified manner.

This can be useful for cleaning up a feature branch before merging, for example.  Note that this does involve modifying existing changesets, so be careful to do it before pushing the changes to another repo.

## 20. Cleaning up

To clean up your working directory and remove files not under version control, you can display a list of untracked files (`-u`) without displaying the status (`-n`), then pipe the results to a command to delete them:

    $ hg st -nu | xargs rm

If you want to clear away all the files in your working directory (but otherwise leave your repository intact) use:

    $ hg checkout null

All tracked files will be removed, leaving your working copy empty (but of course the repository still intact).  This can be useful on servers, in clones, and to save space in older projects.

# Miscellaneous Tips

## 21. Previous References You've Viewed

(This doesn't apply to hg.)

## 22. Branch Naming

Branch names can contain the usual characters, such as a-z, A-Z and numeric characters.  But you can also use other punctuation characters, so you can create a psueo-namespace:

    $ hg branch experimental/parsing_performance

## 23. Finding Who Dunnit

It can be very useful to see who made a particular change in a file.  The `annotate` command will display a file, and annotate each line with the revision and author of the last modification.  (This command is often known as the `blame` command, as you probably want to know who broke the build.)

## 24. Repository Maintenance

The Mercurial repository format ("reflog") is designed to be reliable and efficient.  Unlike the git storage format, it does not require ongoing maintenance (such as garbage collection).

Since file systems and hard drives are fallible, it is possible that an error could corrupt a repository.  To verify the integrity of the repository, run:

    $ hg verify

This will perform an extensive series of tests, and report the results.

## 25. Ignore files by pattern

Your working directory can often become cluttered with generated or intermediate files that you do not want to check in to your repository.  You can add a list of files to ignore, either for all repos (by adding it to your `~/.hgrc`) or selectively for a given repository (by editing `.hgignore` in the root of your repo).  You can use either a simple globbing form, or a full regexp specification.  Here's an example:

    syntax:glob
    *.o
    *.a
    *.dylib
    *.pyc
    *.res
    *~

This tells Mercurial to simply ignore these files.  If you are using Xcode, you will want to exclude the build product tree entirely, and also the per-user project settings (which change frequently and don't really need to be under version control):

    build/*
    MyProject.xcodeproj/*.pbxuser
    MyProject.xcodeproj/*.mode1v3

## 26. Bonus: Built-in Web Server

Thanks to its Python foundation, Mercurial features a built-in web server.  You can browse the revision history, review metadata, display a graphical timeline view, list tags and branches, and view file lists and contents.  It is fully configurable, with different styles and templates.  But to get up and running very quickly, simply run the following command:

    $ hg serve -d -n myproject

The `-d` will daemonise the server, which will run it in the background.  By default, the server listens on port 8000, so simply fire up your browser and point it to:

    http://localhost:8000/

and that's it!  You can run multiple instances, install it alongside Apache, and have it run a webdir serving multiple repositories.  It's very powerful and useful.

# Closing

I hope this is a useful set of tips.  Thanks again to Andy for the original `git` document.  Please leave a comment or drop me an email if you have any suggestions or improvements for the above.
