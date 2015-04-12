
# Git training

By [Brent Beer](https://twitter.com/brntbeer), [Github](https://github.com/brntbeer). Took place on April 8th, 2015.


## Data structure

Commit has:
* tree
* author
* committer
* parent
* message

Tree has:
* trees
* blobs

Blob has:
* Content?

An annotated tag has:
* Tagger
* Message
* Reference to a commit

A non-annotated tag has:
* Reference to a commit


```bash
git init newprojectname

tree .git/objects/
ll .git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
git cat-file -t e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
git log -1 --format=raw acc0c92444f8e45880492c57ca16e83e96b274cc

git cat-file -t a4f7f40bced5647bada2ff675093bc437bd431b4
git cat-file -p a4f7f40bced5647bada2ff675093bc437bd431b4

git show master^{tree}
git show master:

git status -u # (find the config settings for that)

git log -1 --format=raw

git cat-file -p `git rev-parse master:`
git cat-file -p abe96cb00784daf55d04f06c08e960b1b6a48a16 # (one of the trees in it)
```

## Tagging

```bash
git log --oneline --decorate
git tag 0.1-pre
git log --oneline --decorate

git tag 0.1 -m 'tag message'
git log --oneline --decorate
git show 0.1
```

## Filter-branch

```bash
touch .env non-secret
git add non-secret
git commit
git add .env
git commit
git rm .env
git commit

git filter-branch --index-filter 'git rm --cached --ignore-unmatch .env' HEAD

git status -s

# Separate a folder into a different repo

git filter-branch --subdirectory-filter docs/
git status
git remote set-url origin https://github.com/user/newemptyrepo
git push -u origin master
```

## Format patches

```bash
git touch x
git add x
git commit
mkdir scripts
mv x scripts/
git add -A
git commit
git log -- scripts/x
git log --follow -- scripts/x
git log --oneline --decorate --all
git log --oneline --decorate --all -2

git touch z
git add z
git commit

git checkout -b b1
git mv z scripts/
git commit
git format-patch master --stdout > ../change.patch

## elsewhere on the master:
git am --signoff < ../change.patch
```


## rebase

Fork `git@github.com:githubteacher/rebase-example.git`

```bash
git clone git@github.com:danyshaanan/rebase-example.git
cd rebase-example
git config --local alias.lg 'log --oneline --decorate --graph --all'
git lg

git rebase -i 9302ba5
```

```
pick e82e729 Licence (MIT)
squash 3a46bcd Moved to CC BY-NC-SA 3.0 license
pick 924c94d Centered footer
reword 18b8a3c iLink to Github repo
pick 5c789b5 Updated ex2 steps
```

then:
```
# This is a combination of 2 commits.
# The first commit's message is:

added CC BY-NC-SA 3.0 license

# This is the 2nd commit message:

Originally had MIT, changed to CC BY-NC-SA 3.0 license
```
then:
```
Link to GitHub repo
Actual: http://...
```

```bash
git lg
git push --force

git checkout experiment
git rebase --onto master experiment~3 experiment
git status
git push --force
```


### Rerere

Fork `git@github.com:githubteacher/rerere-example.git`

```bash
git clone git@github.com:danyshaanan/rerere-example.git
cd rerere-example
git lg
git checkout features/171-long-lived
git log --oneline --decorate --graph --branches
git config --local rerere.enabled true
git merge master
# resolve
git commit
ll .git/rr-cache
cat .git/rr-cache/*/*

# simulate the same situation with with:
cat .git/ORIG_HEAD
git reset --hard ORIG_HEAD
git reset --hard intermediate-long-lived
git checkout master
git reset --hard final-master
git lg

git checkout features/171-long-lived
git merge master

# js/playground.js had a conflict, but was automatically resolved by the rerere

git add js
# resolve css conflict
git add css
git commit

git reset --hard ORIG_HEAD
git reset --hard final-long-lived

git config --local rerere.autoupdate true

git checkout master
git merge features/171-long-lived

## all resolved!

git commit
```


## Misc

### Links

* [Subtrees](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec)
* [Merge vs Rebase](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa)
* [Splitting a Commit with a rebase](http://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Splitting-a-Commit)
* [bfg-repo-cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
* [tmux](http://tmux.sourceforge.net/) - a terminal cloning program that [might be similar to screen](http://www.wikivs.com/wiki/screen_vs_tmux)
* [Gendi cli](https://github.com/gandi/gandi.cli)

### Points

* Disabling `push --force` is only for enterprise
* No quick answer to the case issues
* Mixing rebase and merge in the same repo should be ok
* Rebase does not rewrites tags, hence they will point to commits that do not lead to the branch
* In vi `:x` is an alias of `:wq`

### Quotes

* He who controls the code controls the universe
