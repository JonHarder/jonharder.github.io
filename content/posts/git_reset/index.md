## Introduction
Starting with this article and continuing for the next handful of posts, we will
more complicated scenarios by _changing the history_ of commits in a repository.

But before we dive in, I want to issue a warning.
If you follow them you should mostly stay clear of becoming another cautionary
tale.
1. Changing history that hasn't been pushed anywhere yet is always safe.
2. Changing history that has been pushed but not pulled down by others is safe.
3. Changing history that others have pulled down is a recipe for _pain_ and
   _sadness_. **Do not do this**.
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