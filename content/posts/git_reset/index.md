+++
date = '2025-01-01T15:40:00-06:00'
draft = false
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
8c38345 (HEAD -> main) Add greet method
88b396a (origin/main) Initial main.go with a simple hello world
```

This is the history of changes from the two commits:

```bash
git log --patch --oneline
```

```diff
bd5e9ea Updated main.go with greet function
diff --git a/main.go b/main.go
index a36de8e..ba963c4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,11 @@ package main
 
 import "fmt"
 
+func greet(name string) {
+	   fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-	   fmt.Printf("Hello world\n")
+	   // fmt.Printf("Hello world\n")
+	   greet("Steve")
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
+	   fmt.Printf("Hello world\n")
+}
```

Now let's say we want to grab the state of `main.go` as it was in the first
commit and apply it on top of the current index.

```bash
git reset 88b396a main.go
```

```bash
Unstaged changes after reset:
M       main.go
```

Let's see what it did, starting with a simple `git status`

```bash
git status
```

```bash
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
-     fmt.Printf("Hello, %s\n", name)
-}
-
 func main() {
-     greet("Steve")
-     // fmt.Printf("Hello world\n")
+     fmt.Printf("Hello world\n")
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
+     fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-     fmt.Printf("hello world\n")
+     // fmt.Printf("Hello world\n")
+     greet("Steve")
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
    // fmt.Println("Hello world\n")
    greet("Steve")
}
```

Ah, so it left the file in the working tree exactly as it should be according to
the latest commit, it only updated the index. So to keep the revised change
created by the index, simply revert the unstaged changes with
`git restore <file>`. But to keep the original copy and throw away the changes
created by reset, revert the changes in the index using
`git restore --staged <file>`.

Resetting files is not the main focus of this article however, so let's move on
to resetting the working tree.

### Resetting the working tree

Let's look at our example again with just the two commits:

```bash
git log --oneline --patch
```

```diff
bd5e9ea (HEAD -> main) Updated main.go with greet function
diff --git a/main.go b/main.go
index a36de8e..ba963c4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,11 @@ package main
 
 import "fmt"
 
+func greet(name string) {
+	   fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-	   fmt.Printf("hello world\n")
+	   // fmt.Printf("hello world\n")
+	   greet("Steve")
 }
88b396a (origin/main) Initial main.go with a simple hello world
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
+	   fmt.Printf("hello world\n")
+}
```

The sharp-eyed reader may spot a minor annoyance on the HEAD commit. It looks
like we accidentally added some unused, commented out code in the commit. That
code shouldn't be there.

Now we could clean this up by making another commit which removes the commented
out line of code, but as you've guessed. Git reset can help us clear up this
problem and leave no trace that this oversight ever happened.

But first, we need to make sure we don't cause a time paradox. The first and
most important rule is:

> Changing history that hasn't been pushed anywhere yet is always safe.

You can tell at a glance from the git log output if a particular commit has been
pushed to a remote by the content between the `()` on the commit message. In
this case you can see the first commit _has_ been pushed because it says:
`(origin/main)`. The second commit however has _not_ been because it lacks any
reference to the `origin`: `(HEAD -> main)`. This tells us this commit is the
HEAD commit and it's on the main branch, and it also tells us it hasn't been
pushed to our origin.

To confirm this via a different method, we could use a more advanced feature of
`git log` by specifying an exclusion to the set of commits to log. Prefixing a
revision range with a `^` means: _remove from the range of commits to log all
commits in this range_.

So if we want to view all the commits on our main branch that AREN'T pushed to
the origin, we can have `git log` them like this:

```bash
git log --oneline --decorate HEAD ^origin/main
```

```bash
bd5e9ea (HEAD -> main) Updated main.go with greet function
```

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

```diff
diff --git a/main.go b/main.go
index a36de8e..ba963c4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,11 @@ package main
 
 import "fmt"
 
+func greet(name string) {
+	   fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-	   fmt.Printf("hello world\n")
+	   // fmt.Printf("hello world\n")
+	   greet("Steve")
 }
```

Now our latest commit is undone, but the file changes from it are present and
unstaged. Let's delete our stray commented code and re-add and commit. _type
type type..._

```bash
git add main.go
git commit
```

```bash
git log --oneline --decorate
```

```bash
8c38345 (HEAD -> main) Add greet method
88b396a (origin/main) Initial main.go with a simple hello world
```

And as for the code in our latest commit:

```bash
git log --oneline --decorate --patch -n 1
```

```diff
8c38345 (HEAD -> main) Add greet method
diff --git a/main.go b/main.go
index a36de8e..75bf4f4 100644
--- a/main.go
+++ b/main.go
@@ -2,6 +2,10 @@ package main
 
 import "fmt"
 
+func greet(name string) {
+	   fmt.Printf("Hello, %s\n", name)
+}
+
 func main() {
-	   fmt.Printf("hello world\n")
+	   greet("Steve")
 }
```

Much better.

Our commit has been fixed, and because this wasn't pushed to our remote before
changing, no one has to know we carelessly committed code that shouldn't have
been there. There is no history of that line in a previous commit and the fixup
commit which resolves it. Our history has been kept clean, and our commits
remain logical, self-contained, testable, and deployable. And because we caught
our mistake before we pushed anything to our remote, there was no risk of
changing history of code that others have already checked out.

However, if you followed along with the steps, you may have noticed we needed to
re-write the commit message after we reset it because that commit was blown
away. That's fairly annoying especially if you spent significant effort in
crafting a [good commit](/posts/git_commit). There is a shorthand way to work
through the steps of this exact scenario of fixing code changed in the HEAD
commit using `git commit --amend`. It's less flexible than `git reset`, but if
that's all you need than it's very slick.

Rewind our conversation to the point that we realized we needed to remove the
errant commented out code. Simply make the edit by deleting that line of code,
then add the change to the index with `git add`. Lastly run `git commit --amend`
and the previous commit message will be opened up again. You can choose to
change the wording if you like, but once you save, the HEAD commit will be
altered with your added change all in one go. No reset required.

> [!WARNING]
> This method _still_ changes history and the contents of your commit, meaning
> the rules to follow about when it's safe to change history still apply.

If you need to make edits to commits earlier in the tree than the current one
(the HEAD commit) than this solution (without any modifications) will not help
you. To deal with that problem, stay tuned for the dedicated articles on
`amend`, `fixup`, `autosquash`, and `interactive rebase`.

That's all you need to know to get started with using `git reset`. You can go on
your merry way resetting your commits.

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
