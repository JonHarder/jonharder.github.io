+++
date = '2024-08-26T16:00:00-06:00'
draft = true
title = 'Git Reset'
show_reading_time = true
tags = ['how-to', 'git']
+++

## Introduction

The [last post](/posts/git_commit) in our series on git discussed the structure
and importance of git commits. Ideally, every commit you make would be clear,
concise, small, logical, and self-contained. Unfortunately, reality rarely
allows us to know ahead of time what exactly will end up becoming a perfect
commit. You may commit the first step in your feature, and come to realize while
you are working on the next step that your first commit is not quite right. Or
maybe you notice a simple typo in the work you just committed.

In either case, the simplest solution to this problem is to simply make the
change necessary to resolve the issue you've discovered with the first commit by
making a second commit. This addresses the issue(s) you found, but do these two
commits individually embody the qualities we discussed in the previous post? If
there was a small typo that caused a syntax error in your application, the
initial commit likely fails testing and would not be deployable. The second
commit which resolves the typo would certainly be small, logical, testable,
self-contained, and deployable, but it feels like an opportunity was missed for
wrapping this up in the first commit.

Starting with this article and continuing for the next handful of posts, we will
discuss the tools git provides us to clean up situations like this and other,
more complicated scenarios by _changing the history_ of commits in a repository.

But before we dive in, I want to issue a warning.

## Don't cause a time paradox

This and the future git operations we will be discussing involve _changing
history_ in git. You may have heard that you should never do this, that it
causes problems, that you will run into inscrutable merge conflicts that
resurface over and over and over. These things are true... _sometimes_, but only
if you're not careful about when and how to change history.

There are a few key principles to keep in mind around changing history in git.
If you follow them you should mostly stay clear of becoming another cautionary
tale.

1. Changing history that hasn't been pushed anywhere yet is always safe.
2. Changing history that has been pushed but not pulled down by others is safe.
3. Changing history that others have pulled down is a recipe for _pain_ and
   _sadness_. **Do not do this**.

The first point is the most important to remember. If you are working on a local
series of commits (or patches) that haven't been pushed anywhere, you can change
the history of those commits however you want. No one else could possibly see
those commits, so no one could be effected by you changing them or even know
that you've changed them. You are the master of your own working tree[^1]

## Git Reset

There are two, very different, functions of the git reset command, and it's
important to understand the distinction between them.

The first form deals with resetting the states of **files**, and the second form
deals with resetting the state of the working tree. The former doesn't
technically _change_ history as it does not affect the working tree at all,
therefore it will only be covered briefly in this article. The latter form
**DOES** change history and will be the primary focus of this article.

### Resetting files

The git help docs for `git-reset` have this to say about resetting files:

> These forms reset the index entries for all paths that match the \<pathspec\>[^6]
> to their state at \<tree-ish\>[^7]. (It does not affect the working tree or
> the current branch.)

This means we can set all objects referenced by the _pathspec_ to the state they
were at the referenced point in history according to `tree-ish`. Since we've
used the time travel analogy, this is like going back in time and pulling a
friend from a year ago into your time machine and bringing them back to the
present day, replacing the present day friend with the past one.

#### Example

Given a simple repo with two commits:

```
8a426ec (HEAD -> main) Updated main.go with greet function
88b396a Initial main.go with a simple hello world
```

This is the history of changes from the two commits:

```bash
git log --patch --oneline
```

```diff
8a426ec (HEAD -> main) Updated main.go with greet function
diff --git a/main.go b/main.go
index a36de8e..75bf4f4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,10 @@ package main

 import "fmt"

+func greet(name string) {
+       fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-       fmt.Printf("hello world\n")
+       greet("Steve")
 }
88b396a Initial main.go with a simple hello world
diff --git a/main.go b/main.go
new file mode 100644
index 0000000..a36de8e
--- /dev/null
+++ b/main.go
@@ -0,0 +1,7 @@
+package main
+
+import "fmt"
+
+func main() {
+       fmt.Printf("hello world\n")
+}
```

Now let's say we want to grab the state of `main.go` as it was in the first commit
and apply it on top of the current index.

```bash
git reset 88b396a main.go
```

```
Unstaged changes after reset:
M       main.go
```

Let's see what it did, starting with a simple `git status`

```bash
git status
```

```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   main.go

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   main.go
```

Hmmm. The index was clean before performing the reset, what are the staged
changes? And why are there unstaged changes too? Let's take a look.

```bash
git diff --staged
```

```diff
diff --git a/main.go b/main.go
index 75bf4f4..a36de8e 100644
--- a/main.go
+++ b/main.go
@@ -2,10 +2,6 @@ package main

 import "fmt"

-func greet(name string) {
-       fmt.Printf("Hello, %s\n", name)
-}
-
 func main() {
-       greet("Steve")
+       fmt.Printf("hello world\n")
 }
```

That makes sense. These would be the changes to take our current `main.go` back
to the state it was in the first commit.

What are the unstaged chagnes?

```bash
git diff
```

```diff
diff --git a/main.go b/main.go
index a36de8e..75bf4f4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,10 @@ package main

 import "fmt"

+func greet(name string) {
+       fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-       fmt.Printf("hello world\n")
+       greet("Steve")
 }  
```

This diff looks like it would undo the reset. Remember, these are the unstaged
changes. This is the current state of the file, so we can just `cat` it to see
what it looks like.

```go
package main

import "fmt"

func greet(name string) {
        fmt.Printf("Hello, %s\n", name)
}

func main() {
        greet("Steve")
}
```

Ah, so it left the file in the working tree exactly as it should be according
to the latest commit, it only updated the index. So to keep the revised change
created by the index, simply revert the unstaged changes with `git restore <file>`.
But to keep the original copy and throw away the changes created by reset, revert
the changes in the index using `git restore --staged <file>`.

Resetting files is not the main focus of this article however, so let's move on
to resetting the working tree.

### Resetting the working tree

Let's look at an example:

```bash
git show HEAD
```

```
commit edaa773a1f2db10254ed79cf015c40b515fbb92c
Author: Jon <jharder@kipsu.com>
Date:   Wed Aug 28 12:08:35 2024 -0500

    Add additional functionality to my-feature in file2

diff --git a/file2 b/file2
index 73d3ef3..9d24812 100644
--- a/file2
+++ b/file2
@@ -1 +1,3 @@
-This is my file.
\ No newline at end of file
+This is my file.
+
+Additional line which enhances my-featue.
\ No newline at end of file
```

The sharp-eyed reader may spot the issue with this commit. On the second line of
the addition to `file2`, we made a typo with `my-featue` instead of
`my-feature`. We know we want to fix the typo, but how do we do it?

One option we have is `git reset`.

## Git reset

Git reset[^2] allows you to "un-commit" a commit, giving you effectively an
"undo" if you realize you weren't quite finished with your changes.

Going back to our example from before, this is the state of our working tree:[^3]

```bash
git log HEAD ^main
```

```
commit faa66ffce98ea84be2fa61497799ca72b1f64c85
Author: Jon <jharder@kipsu.com>
Date:   Thu Aug 29 12:00:01 2024 -0500

    Add additional functionality to my-feature in file2

commit 5c4472811dca6b57b7cc2aba4460d335c72d444a
Author: Jon <jharder@kipsu.com>
Date:   Thu Aug 29 11:58:52 2024 -0500

    Introduce file2 for my-feature

    This adds file2 which allows my-feature to work in a minimally viable state.
```

These are the two commits on our `my-feature` branch that aren't in the `main`
branch.

If we try to see what commits in our branch have been deployed to our remote
(called `origin` in this repo), we can see this branch hasn't been pushed at
all.[^4]

```bash
git log HEAD ^origin/my-feature
```

```
fatal: bad revision '^origin/my-feature'
```

This means there is no branch `my-feature` pushed to our remote called `origin`.

So, if we think back to our rules about when you can safely change history, we
satisfy the first and most important rule: _"Changing history that hasn't been
pushed yet is always fine."_

Now then, let's get to the point of this whole article...using `git
reset`.

In the following example, we want to reset our repository to the state it was at
in the previous commit.[^5]

```bash
git reset HEAD~1
```

But what about the files changed in this commit? What's going to happen to them
when we `reset`?

By default, git will perform a `--mixed` reset, meaning the file changes will be
retained, but will be removed from the index (where files go when you `git add`
them). The other options you could specify are `--soft` which keeps the changes
in the index and `--hard`, which removes them from the index AND the working
tree. With `git reset
--hard`, the changes made in your commit will be completely
erased.

After the reset, the changes from the latest commit will be in the working tree,
but not added to the index:

```bash
git diff
```

```
diff --git a/file2 b/file2
index 197d1d0..256b425 100644
--- a/file2
+++ b/file2
@@ -1 +1,3 @@
 This is my file.
+
+Additional line which enhances my-featue.
```

It's here that we can see our typo ("my-featue."). Let's fix that.

_type type type..._

```bash
git diff
```

```
diff --git a/file2 b/file2
index 197d1d0..d0adfb9 100644
--- a/file2
+++ b/file2
@@ -1 +1,3 @@
 This is my file.
+
+Additional line which enhances my-feature.
```

Much better.

Now, we just simply commit the change again.

```bash
git add file2
git commit -m 'Add additional functionality to my-feature in file2'
```

```
[my-feature 3114a56] Add additional functionality to my-feature in file2
 1 file changed, 3 insertions(+), 1 deletion(-)
```

```bash
git log -n 1 -p
```

```
commit 3114a56e6cff008d07081e9004aa0b24d10df7ae
Author: Jon <jharder@kipsu.com>
Date:   Wed Aug 28 11:56:04 2024 -0500

    Add additional functionality to my-feature in file2

diff --git a/file2 b/file2
index 73d3ef3..2bfb629 100644
--- a/file2
+++ b/file2
@@ -1 +1,3 @@
-This is my file.
\ No newline at end of file
+This is my file.
+
+Additional line which enhances my-feature.
\ No newline at end of file
```

There we go. Our commit has been fixed, and because this wasn't pushed to our
remote before changing, no one has to know we had a typo. There is no history of
the typo in an previous commit and the fixup commit which resolves it. Our
history has been kept clean, and our commits remain logical, self-contained,
testable, and deployable. And because we caught our mistake before we pushed
anything to our remote, there was no risk of changing history of code that
others have already checked out.

That's all you need to know to get started with using `git reset`. You can stop
here and go on your merry way resetting your commits.

If you're hungry for more, stick around for some bonus, advanced content.

## Bonus contest

Still here? Cool. Let's talk about some of the other, more nuanced uses of
`git reset`.

We are going to cover two scenarios that cover the concept of selectively
resetting changes from the latest commit. First we will discuss removing files
from the commit, and second we will discuss removing specific hunks (selections
of changes to multiple, adjacent lines in a file) from a commit.

### Removing a file

Say you were working on some changes that spanned a series of files. You finish
up your work, do a quick `git commit --all` to add and commit all changed files
then push your work to your remote. Only then do you realize you had some
unrelated, in progress work that got slurped in with the `--all` flag.

```bash
git show HEAD
```

```
commit 36dfb3977572330fb042dedc83f9654a99a6cfcf
Author: Jon <jharder@kipsu.com>
Date:   Wed Aug 28 15:22:23 2024 -0500

    Add configuration files for production and stage.

    This allows us to store separate application configuration for our
    production and staging environment.

diff --git a/config/production.ini b/config/production.ini
new file mode 100644
index 0000000..e69de29
diff --git a/config/stage.ini b/config/stage.ini
new file mode 100644
index 0000000..e69de29
diff --git a/unfinished b/unfinished
new file mode 100644
index 0000000..e69de29
```

You can see given the commit message and the files changed that
`config/production.ini` and `config/staging.ini` likely belong in this commit.
`unfinished` however...probably doesn't belong.

We _could_ reset the whole commit, remove `unfinished` from the index, then
re-commit our code, but `git reset` offers a more efficient way.

```bash
git reset --patch HEAD~1
```

```
diff --git b/config/production.ini a/config/production.ini
deleted file mode 100644
index e69de29..0000000
(1/1) Apply deletion to index [y,n,q,a,d,?]? n

diff --git b/config/stage.ini a/config/stage.ini
deleted file mode 100644
index e69de29..0000000
(1/1) Apply deletion to index [y,n,q,a,d,?]? n

diff --git b/unfinished a/unfinished
deleted file mode 100644
index e69de29..0000000
(1/1) Apply deletion to index [y,n,q,a,d,?]? y
```

Let's take a look and see what that did

```bash
git status
```

```
On branch main
Your branch is ahead of 'origin/main' by 3 commits.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
    deleted:    unfinished

Untracked files:
  (use "git add <file>..." to include in what will be committed)
    unfinished
```

Git reset has a secondary function, unrelated to undoing commits. In this usage,
it changed a file back to its state in the previous (`HEAD~1`) and staged (added
to the index) the file modification necessary to make that happen. In our case,
since we created a new file, `unfinished`, it deleted the file from git's
working tree, making it untracked. The file still exists on the file system, but
git won't know anything about it.

One important thing to note here is that in this case, **we didn't modify the
commit contents**.

```bash
git show HEAD
```

```
commit 340207fe180fbc54dc74fb9edcdee2cf6c3eaad3
Author: Jon <jharder@kipsu.com>
Date:   Wed Aug 28 15:22:23 2024 -0500

    Add configuration files for production and stage.

    This allows us to store separate application configuration for our
    production and staging environment.

diff --git a/config/production.ini b/config/production.ini
new file mode 100644
index 0000000..e69de29
diff --git a/config/stage.ini b/config/stage.ini
new file mode 100644
index 0000000..e69de29
diff --git a/unfinished b/unfinished
new file mode 100644
index 0000000..e69de29
```

As you can see, `unfinished` is still in the head commit. To absorb this change
(deleting `unfinished`) into that commit, we can use a feature of `git commit`,
called `amend`. This takes your changes in the index, and **replaces** the
`HEAD` commit with a new one based off of the current state of the index. We
also used the `-C` flag, short for `--reuse-message` to reuse the author and
commit message of the commit we are going to replace.

```bash
git commit --amend -C HEAD
```

```
[main 081fca1] Add configuration files for production and stage.
 Date: Wed Aug 28 15:22:23 2024 -0500
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 config/production.ini
 create mode 100644 config/stage.ini
```

Now, as we can see, the file is gone (according to git).

```bash
git show HEAD
```

```
commit 081fca1353fe285882f6e02dce0fc02b9b7eaf1d
Author: Jon <jharder@kipsu.com>
Date:   Wed Aug 28 15:22:23 2024 -0500

    Add configuration files for production and stage.

    This allows us to store separate application configuration for our
    production and staging environment.

diff --git a/config/production.ini b/config/production.ini
new file mode 100644
index 0000000..e69de29
diff --git a/config/stage.ini b/config/stage.ini
new file mode 100644
index 0000000..e69de29
```

But it still exists on disk

```bash
ls -l
```

```
total 8
drwxr-xr-x@ 4 jharder  staff  128 Aug 28 15:22 config
-rw-r--r--@ 1 jharder  staff    0 Aug 27 15:49 file1
-rw-r--r--@ 1 jharder  staff   60 Aug 28 15:01 file2
-rw-r--r--@ 1 jharder  staff    0 Aug 28 15:22 unfinished
```

We introduced a lot in this bonus section. Amending commits will be covered more
fully in a future article in this series, so don't worry if it didn't all quite
make sense.

## Additional posts in this series

- [What makes a good commit](/posts/git_commit)
- [git reset](/posts/git_reset) _(this post)_
- amend _coming soon_
- fixup
- interactive rebase
- I've made a time paradox, now what?

## Footnotes

[^1]: <https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefworkingtreeaworkingtree>

[^2]: <https://git-scm.com/docs/git-reset>

[^3]: The syntax here tells git to show all commits in the repo\'s history
    starting with `HEAD` (the current, most recent commit), but subtracting all
    the commits that are present in `main`. This is the purpose of the `^`
    prefix when specifying a range of commits in `git
    log`. This has the
    effect of showing what commits on this branch haven\'t been integrated into
    `main` yet.

[^4]: This is similar to the previous `git log` command, but this time, we want
    to view all the commits in history starting with the current `HEAD` commit
    in our local working tree, but exclude any that have been pushed to
    `origin/my-feature`, meaning commits that have been pushed to github into
    the `my-feature` branch. Another way we could phrase this is: what commits
    do I have locally on this branch that I haven\'t pushed yet?

[^5]: `HEAD` specifies the current tip of the history in git. This means the
    most recent commit. `~` signifies a _reverse_ traversal, or going backward
    in time. Therefore, `HEAD~1` means: _the previous commit_.

[^6]: A **pathspec** is a way to restrict which files a git command should work
    on. The simplest form of a pathspec is a file or directory eg.
    `git add src/` or `git log README.md`. But there is more to pathspecs than
    just simple paths. A fuller dive into the intricacies will be reserved for a
    future article.

[^7]: A **tree-ish** is anything that points to a particular point in your
    repo's git history. A commit points directly to a particular point in the
    tree of your history. A tag can be dereferenced (following the tag to the
    thing it points to) to a commit (or the HEAD of a branch). Think of it as
    any object that points to a particular point in the tree of commits.
