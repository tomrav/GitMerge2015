
# Git basics and data structure

Learning Git basics and structure

## What is Git?

Git is a Version Control system - a way to manage the changes and progress of a project, so that:
* We'd know when each change was made, by whom, and why.
* We'd be able to revert specific changes, even if they were done a while ago.
* We'd be able to return to previous states of the project.
* We'd be able to work on the same project at the same time, without being constantly in sync.

## How does Git work?

Git offers an advanced way to click the "save button" on a project.
Each "save" is an independent snapshot of the project, which is added to the previous ones instead of overriding them.
Each such "save" is called a "commit", and contains an author, a description, a timestamp, and a complete snapshot of the project.

## Lets start

While there are lots of tools to handle the Git actions, we'll work with the most basic one - the command line interface.
This will allow us to best understand the git commands and workflow.
This assumes an understanding of terminal commands such as `mkdir`, `cd`, `ls`, `echo` and `cat`.

### Initializing a new git repo

A Git repo (repository) is a directory which is tracked by Git.
We can create a new repo by using `git init` in the directory we want tracked.
Lets create a new directory, and initialize Git in it:

```bash
mkdir myRepo
cd myRepo
git init
# Initialized empty Git repository in /Users/dany/myRepo/.git/
```

Notice that? Git has created a `.git` directory inside the 'myRepo' directory. We'll look into it a bit later.

We can always ask Git for the current repo's status with `git status`.
If we'll run it now, Git will tell us that there is "nothing to commit", so lets add something to commit.

### Our first commit

We can create a file with the content "hello" by echoing the "hello" string straight into a file, (even if it doesn't yet exist), with:

```bash
echo 'hello' > file.txt
```
If we will now run `git status`, git will tell us that `file.txt` is untracked, meaning that it's a new file for git.
In order to include it in our next commit (which is our "save"), we need to add it with:
```bash
git add file.txt
```
Running `git status` now will tell us that the new file `file.txt` is set to be committed.
We can commit it with:
```bash
git commit --message 'hello file created'
```

`git log` will show us the log, currently containing only one commit.

### So, what's inside the commit?

Before we'll look inside the commit, we need to understand what a SHA is:

A SHA is a string of length 40, which includes only numbers and the letters a-f.
In Git, each SHA represent a "git object", which is an atomic unit of information git keeps.
There are a few types of git objects, which we will see shortly.

So, what's inside the commit?

In order to check that, we need to run `git cat-file -p` on the commit's SHA.
Get the commit's SHA from the first line of your `git log`, and fill in the command:

```bash
git cat-file -p <SHA_OF_COMMIT>
```

We can see that our commit has a tree, author, committer, and a message:

```bash
#tree e31a96220fbfbe7601ecc086a36b96dc27a8867e
#author Dany Shaanan <danyshaanan@gmail.com> 1430053260 +0300
#committer Dany Shaanan <danyshaanan@gmail.com> 1430053260 +0300

#hello file created
```

Notice the long string after "tree"?
That's a SHA, meaning that 'tree' is another git object.
While a commit represent a snapshot of the whole project, a tree represent a snapshot of one directory, and all of its contents.

Lets look at the tree, again, using `git cat-file -p` on its SHA:

```bash
git cat-file -p e31a96
#100644 blob ce013625030ba8dba906f756967f9e9ca394464a	file.txt
```

It seems that the tree contains a blob, which also has a SHA, which means it is also a git object!
If we'll `git cat-file -p` on the blob's SHA, we'll see the actual content of `file.txt`.

```bash
git cat-file -p ce0136
#hello
```

We can understand from this that a blob represents a file.

So we see that a commit has some data, including a reference to a tree, that includes a blob that represents a file and contains text.
All together, this is a full description of the project's snapshot that is the commit. All of the data is right here!!

(And if we'll look into our `.git/objects` folder, (like with `find .git/objects -type f`), we'll see that everything is there - the commit, the tree, and the blob).

Let's make another commit and explore it a bit:

```bash
mkdir a
echo 'hello' > a/y
echo 'world' > a/z
git add a
git commit --message 'added hello and world files'

git cat-file -p SHA_FOR_LAST_COMMIT
```

In addition to everything we've seen in the first commit, this one also has a parent - a reference to the previous commit.

Let's look at the commit's tree:

```bash
git cat-file -p b944b4
```

The tree has two objects - the old blob representing the `x` file, and an additional tree, representing the `a` directory! Let's look what's inside:

```bash
git cat-file -p f82a6a
```

The tree representing `a` has two objects - two blobs, for `y` and for `z`.
Notice that the blob representing `z` is the same one representing `x`!
This is not a result of the fact that we already had such a blob in the repo - go into any repo and commit a file with the content 'hello', and you'll get a blob with the SHA starting with `ce0136`. Verify it by running `git cat-file -p ce0136`

### When do we want to commit?

So far, we've created small commits as examples, but in a real repository,
and especially when working with other people, we'd want to create commits which will be:
* Well defined - The commit message should clearly state what was the point of the commit.
* Atomic - The commit should do one thing - refactoring, development, bug fixes, file renaming - should be separated to different commits,
so that each commit could be easily read and understood, and if needed, easily reverted.


## Branches (and HEAD)

So we've seen that during our work, we've created a chain of commits,
but what happens if we want to test something outside of the formal progress of our project, and how can we use it if the test was successful?
(If we'll go back to the old 'save' model, what if we want to handle two versions of the project, and maybe join them later on?)

That's why we have branches.

### What is a branch?

Without noticing, we were already working on one branch - which is by default, named `master`.
We can see this by running `git branch`:

```bash
git branch
# *master
```

Let's see how git knows that we're working on the `master` branch, and what that branch actually is;

Git references the current branch in the HEAD pointer, which is represented by a file located in `.git/HEAD`. We can check what's in it with:

```bash
cat .git/HEAD
# ref: refs/heads/master
```
This tells us that the HEAD is a reference. What to? To `refs/heads/master`, which is a file path inside our `.git` directory, which represents the `master` branch.

Lets use that information to find out what the master branch really is:

```bash
cat .git/refs/heads/master
# 1f6125df54cf55399e5285e287283c584b6e9ce9
```
So the `master` branch is a reference as well.
Since its file contains a SHA, we can deduce that it's a reference to a git object.
We can find out what kind of git object with:
```bash
git cat-file -t 1f6125d
# commit
```

So a branch is a reference to a commit, and the HEAD tells us which branch we're on.

### Creating and switching branches

Since a branch is just a pointer to a commit, creating a branch will do nothing more than add a new pointer to our current commit.
We can create a new branch with `git branch NAME`, and see what in it:
```bash
git branch dev
cat .git/refs/heads/master
cat .git/refs/heads/dev
```

Since we are on the `master` branch, the newly created `dev` branch points to the same place.
Notice that we only created a branch, but haven't "moved" to it, meaning that the HEAD still points to master.
We can change that with the `git checkout` command:
```bash
cat .git/HEAD
git checkout dev
cat .git/HEAD
```
The checkout changed the HEAD, and since both master and dev points to the same commit, nothing else was changed.


### How and when do branches change?

A branch's purpose is to grow, and weather we're in the `master` branch or in a `dev` branch,
the branch should follow us as we progress and add additional commits.
Therefore, whenever we commit, (add a new commit), the current branch is updated to point to the newly created commit.

Currently, our two branches point to the same commit:

```bash
cat .git/refs/heads/master
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
cat .git/refs/heads/dev
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
```

Lets create a new commit:

```bash
echo 'testing testing' > test.txt
git add test.txt
git commit -m 'added test.txt'
```

...and see what happens to our branches:

```bash
cat .git/refs/heads/master
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
cat .git/refs/heads/dev
#ec0668588f94a9322ecdf18df93ef6035eb298bb
```

We can see that the `dev` branch has progressed.
It points to the new commit, which, as we know, represent a snapshot of the whole project.

As before, we can switch branches, and go back to `master`:

```bash
git checkout master
ls -l
```
We can see that checking out the master branch made `test.txt` disappear!
That's by design. Checking out a branch makes sure that our working directory contains the snapshot of the commit the branch points to.
We can get back to the `dev` branch and see `test.txt` again:
```bash
git checkout dev
```

To summaries:
* HEAD is a pointer to a branch, so it changes when we switch a branch (with `git checkout`).
* A branch is a pointer to a commit, and it changes when it's the current branch (HEAD points to it) and we add a new commit.


## Syncing our repository

So far, we've only worked locally. Unlike with some other version control systems,
we were able to commit code without using our network connection.
This is possible because in git, each user has the whole repository,
including all history and all branches.

Now, let us share our code.

### Remotes

Each repository has "remotes", which are the remote places with which the local repo can sync.
Since we've created a new repo locally, (and not "cloned" it from a remote), it doesn't have any yet.
We can see that with:
```bash
git remote -v
```
No remotes!

Since this is a new repository, we're want to sync it up to an empty repo.
We can create an empty repo in Github.
After we'll do that, Github will tell us that we can add a new remote with:
```bash
git remote add origin git@github.com:username/reponame.git
```
What we've done here is to tell git to add a new remote, called 'origin', which points to that git url.
We could now see that `git remote -v` tells us that this remote is set up.

### Push and Pull

Now that we have an empty remote waiting for us, we can do out first push there.
We'll push master to the remote, but on the first time, we'll have to tell git where exactly in the remote we want to push it.
We should make sure that we're on master with:
```bash
git checkout master
```
and push it with:
```bash
git push --set-upstream origin master
```
This is telling git to associate our current branch, with the remote branch on origin called master, and push it.
This is done so that on the next time we'll want to push, we could just `git push`, and git would know where we want to push master to.

Once we've done that, and once there are remote changes,
(which were done by someone else, or by us from a different computer),
we could `git pull` to get the remote changes.













///////////////////////////////////////////////


## Notes

* There are 4 types of objects: commit, tree, blob and tag (which we haven't covered here).
 * A commit object contains a parent (commit), an author, a committer, a message, and a tree.
 * A tree object contains blobs and trees
 * A blob object contains text data.
 * A tag object contains a name, a commit, and a tagger.
* There are 3 types of file pointers:
 * A branch file points to a commit, and is located in `.git/refs/heads/`*
 * The HEAD file points to a branch or to a commit, and is located at `.git/HEAD`
 * A tag file points to a tag object or to a commit, and is located in `.git/refs/tags/`
* A commit represents a snapshot.
* `master` is just a name for the default branch.
* Any action that can be done on a commit, can be done on HEAD, a tag, or a branch, as all those are eventually pointers to commits.
* You can find the type of any object with `git cat-file -t SHA`

*Remotes not discussed here

## Questions

* What happens to existing commits, trees and blobs upon the creation of new commits?
* What is modified when we commit?
* what objects are created when a file renaming is committed?
* What's the difference between `author` and `committer`?
* How can a branch be changed besides by committing while it's checked out?
* How are SHA values calculated for the different types of objects?



<!-- -->

## TODO & metadata
* commands covered: init, add, commit, status, log, branch, checkout (of branches), cat-file
* terms covered: branch, HEAD, commit,
* commands not covered: pull, push, merge, rebase, checkout (of files), revert...
* terms not covered: tag, remote, conflict...
