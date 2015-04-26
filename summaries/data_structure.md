
# Git data structure

An overview of what commits, branches, and tags really are.

## Commits (and trees and blobs)

Lets create a repository with a commit, and see where that commit takes us:

```bash
git init commitsEtc && cd commitsEtc
echo 'hello' > x && git add x && git commit -m x
```

We'll use `git cat-file -p` on any SHA to explore its contents.

```bash
git cat-file -p SHA_FOR_LAST_COMMIT
```

We can see that our commit has a tree, author, committer, and a message:

```bash
#tree e31a96220fbfbe7601ecc086a36b96dc27a8867e
#author Dany Shaanan <danyshaanan@gmail.com> 1430053260 +0300
#committer Dany Shaanan <danyshaanan@gmail.com> 1430053260 +0300

#x
```

Lets look at the tree, again, using `git cat-file -p` on its SHA:

```bash
git cat-file -p e31a96
#100644 blob ce013625030ba8dba906f756967f9e9ca394464a	x
```

This tree contains a blob:

```bash
git cat-file -p ce0136
#hello
```

So we see that a commit has some data, including a reference to a tree, that includes a blob that represents a file and contains text.
If we'll look into our `.git/objects` folder, (like with `find .git/objects -type f`), we'll see that everything is there - the commit, the tree, and the blob!

Let's make another commit and explore it a bit:

```bash
mkdir a
echo 'hello' > a/y
echo 'world' > a/z
git add a
git commit -m a

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




## Branches and tags (and HEAD)

Let's create a repository and see what happens when we commit, create or change branches, and create tags.

```bash
git init branchesEtc && cd branchesEtc
echo 'hello' > x && git add x && git commit -m x
```

Let's look at the `.git/refs/heads/` directory before and after creating a branch:

```bash
ls -1 .git/refs/heads/
#master

git branch dev

ls -1 .git/refs/heads/
#dev
#master
```

The `.git/refs/heads/` directory contains a file for each branch. Let's look at them:

```bash
cat .git/refs/heads/*
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
```

They are both just references to commits!

But if both are identical, how does git know which one we're on?

```bash
cat .git/HEAD
#ref: refs/heads/master
git checkout dev
#Switched to branch 'dev'
cat .git/HEAD
#ref: refs/heads/dev
```

So the `.git/HEAD` file is a pointer to our current branch.
What happens when we commit?

```bash
echo 'world' > y && git add y && git commit -m y

cat .git/HEAD
#ref: refs/heads/dev

cat .git/refs/heads/*
#bb775fe605887f07190bacc6e58fbfd0c56a7a67
#6aba1ca400115f031c36c106fb2c138ea2f4bff3
```

The HEAD doesn't change, of course, but the branch it's pointing to does!

So the HEAD is a pointer to the dev branch, and the dev branch is a pointer to a the `bb775f` commit.
The head changed when we checked out the dev branch, and the dev branch changed when we made a commit while it was checked out (pointed to by the HEAD).


So what's a tag?

```bash
git tag 0.1
cat .git/refs/tags/0.1
#bb775fe605887f07190bacc6e58fbfd0c56a7a67
git cat-file -t bb775fe605887f07190bacc6e58fbfd0c56a7a67
#commit
```

A tag is also a pointer to a commit!
So what's the difference between a tag and a branch?
Both are pointers to a commit, and both can be checked out.
Let's see what happens to the HEAD when a tag is checked out:

```bash
git checkout master
cat .git/head
#ref: refs/heads/master
git checkout 0.1
#(A long message, explained below)
cat .git/head
#bb775fe605887f07190bacc6e58fbfd0c56a7a67
```

HEAD can't point to a tag, and a tag can't be updated by committing, like a branch does.
(Tags are supposed to be constants).


What happened here is that HEAD does not point to a branch any more, but to a commit.
This is called 'detached HEAD' state,
as HEAD is not connected to any branch.
HEAD itself will update upon commits,
but your location will be lost once `git checkout` will be used.
(To prevent that, mark your current location with a new branch using `git checkout -b new_branch_name`).


What's an annotated tag?

```bash
git tag 0.2 -m wat
cat .git/refs/tags/0.2
#448f34f0bb710200615b1f1256282ce4de341fee
git cat-file -t 448f34f0bb710200615b1f1256282ce4de341fee
#tag
git cat-file -p 448f34f0bb710200615b1f1256282ce4de341fee
#object 6aba1ca400115f031c36c106fb2c138ea2f4bff3
#type commit
#tag 0.2
#tagger Dany Shaanan <danyshaanan@gmail.com> 1430054541 +0300
#
#wat
```

A simple tag points directly to a commit object,
and therefore has no extra data of its own.
When creating a tag with a message, a tag object is created,
which contains extra data, including the message, and the commit it points to.


## Notes

* You can easily review the tree of the last commit with `git cat-file -p master^{tree}`.
* You can find the type of any object with `git cat-file -t SHA`
* `master` is just a name for the default branch.
* Any action that can be done on a commit, can be done on HEAD, a tag, or a branch, as all those are eventually pointers to commits.
* There are 4 types of objects: commit, tree, blob, tag.
* Branches and tags are pointers to commits.
* HEAD is a pointer to a branch or to a commit.

## Questions

* Why does `find .git/objects -type f` returns very few results in my very big repo? What are pack files?
* What happens to existing commits, trees and blobs upon the creation of new commits?
* What's the difference between `author` and `committer`?
* What does `git checkout HEAD` do?
* What changes when we commit? What's added?
* How can a branch change besides committing while it's checked out?
* How are SHA values calculated for the different types of objects?



<!-- -->
