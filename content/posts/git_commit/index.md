+++
date = '2024-08-13T16:00:00-06:00'
draft = false
title = 'What Makes a Good Git Commit'
show_reading_time = true
tags = ['how-to', 'git']
+++

# Introduction

I want to kick off this series of articles on git with the humble commit.
Commits are the basic unit of change in your codebase. Like an atom, you cannot
break things down any further than a commit[^1]. Git automatically keeps track
of which files were changed, who changed them, and when. It is up to YOU,
however, to decide what information is added to that commit to give context to
others or your future self about the changes.

To expand on this, let's break down the two (user definable) attributes of a
commit. When we make a commit, we get to say what **message** is attached to it,
and what **changes** to include. We'll start with what makes a good commit
message, and then move on to considering how to structure your changes by
deciding which file changes should be included in one commit vs another.

## What makes a good commit message

### The message heading

Writing a commit message is a time for you to step back and _think_ about the
changes you've made. What problem were you trying to address by making these
changes? What is expected outcome in the application as a result of those
changes? Git can tell you via the diff that you
_`made a function called
'get_latest_foo()' in foo_handler.rs`_, only the mind
that wrote the function can say _why_ you wrote `get_latest_foo()`.

If you've ever tried to debug a problem and walk through the commit history to
find when a bug was introduced, you may have experienced the pain of lousy
commit messages. `Changed some stuff` does not spark joy. `Updated Controller`,
though slightly more descriptive, still does not help the reader to understand
anything about the change beyond _what_ was changed.

It is critical to recognize that commits are more than just a way to back up
your work onto a remote server (e.g. GitHub). Commits are a developer's first
and most valuable source of clues as to why the application is behaving the way
it is. Their efforts can either be aided by a helpful commit message like
_`Move foo retrieval logic to dedicated handler`_ or hindered by a message like
_`wip updates`_.

The header should be short (~50 characters according to Linus Torvolds, the
creator of git) though this is not a hard requirement. It should be short enough
to grok quickly, and state in the imperative[^2] mood what was changed.

### The message body

If you have a good heading on your commit message, you're good right? One could
argue this the header (the first line of the commit message) is all that is
necessary in a commit message, after all, the first line is all you would see in
the standard git log, but the body of a commit provides a great place to
describe in full prose the intent of your change.

When adding a body, you are required to have at least (but preferably only) one
blank line between the header and the body:

    This is the heading

    This is the body of the message. I'm going to put much more content
    here since I have more room here.

This is the place for you to provide all the essential context around why this
change was made. Why was this change necessary? Is this being done in
preparation for any future changes? What is the practical effect on the
application/system as a result of this change? If the changes you made seem a
bit odd at first glance (for example, not re-using existing functionality that
accomplishes your task but instead implementing it from scratch) this is a great
place to explain why the obvious approach was not suitable and why this approach
is better.

You are basically trying to mind read your (or your coworker's) future mind when
they are looking back at your code. This may seem like an impossible task, but
with practice as well as seeing examples of great commits (and even not-so-great
commits) you will get a sense of what information is useful.

Lastly when you have useful commit messages, this information will be used by
your code review system by pre-populating the review message. This saves time by
adding the necessary context you would have added to the pull request anyway,
_and_ has the added benefit of persisting beyond the pull request because it
lives in your code.

## Properties of a good commit (content)

Now that we've covered good commit messages, we'll turn our attention to what
qualities make for a good commit content. The commit message gives the viewer
helpful context as to why a change was made, but _what_ changes go into a commit
are important to consider as well.

Many of the following qualities are born out of the necessity in software
development to inspect commits to troubleshoot a bug, mainly by reverting the
code base back to some previous point in its history or to identify when and why
a change was introduced. This has implications for how to craft both the commit
message and the contents of the commit. Because commit messages have already
been handled above, we will focus just on the ways that commit contents affect
these requirements.

### Logical

Similar to the single responsibility principle[^3], commits should contain
_logically_ related changes. If you are trying to determine if a particular
commit broke a feature, it can be challenging to test when an unrelated change
in that commit causes other problems that cannot be pulled apart. A commit
containing changes to your data model should not also contain changes to fix a
bug in the UI.[^4]

To get a sense of what this looks like, when reviewing a pull request, try to
take a look at the commits within the branch (if you use branch-based
development) one at a time, moving forward through them to see how the feature
was assembled in each step. If the author was disciplined in the creation of
their commits, you will get to see their "proof of work"[^5] in how they added
each part of the feature.

If the author was NOT disciplined (or does not yet know the value of commits
that follow these principles) you will instead become more confused as you try
to follow the flow of changes where one commit introduces a change, the next
commit modifies that change, and a third removes the change entirely. In these
cases you will be better off going back to just looking at the changes made at
the branch level rather than looking at each commit individually.[^6]

### Small

Commits should be _small_. The definition of _small_ can be somewhat hard to
define, but generally a developer will know a **big** commit when he or she sees
one. This comes back to the ultimate goal of making commits understandable by
someone who is reviewing them. This review could be in a formal "code review"
sense in the form of a pull request, or in a triage sense where someone is
trying to find the source of a problem in the system by looking at the git log.
Our brains can only hold so much in working memory at a time, and anything
beyond it drastically reduces comprehension.

If you've ever been asked to review a pull request with 50 files changed, you
have an idea of the difficulty of reviewing big commits.

### Self-contained

Closely related to the above is the quality of being self-contained. A commit
should not (baring undiscovered bugs) leave the system in a broken or otherwise
inoperable state.

Note that this is not the same as the notion of being complete. A commit which
introduces a new function that is never called is self contained, even though
the feature that will eventually utilize that function is not complete. You
(hopefully) should not break anything by adding a new function to a code base,
in fact, it's even deployable!

This may feel counter-intuitive, but this quality helps to leave the code base
in a deployable state at all times. When every commit is self contained, you can
test your work in smaller chunks, and when you can do that, you can pinpoint the
exact commit that introduced an issue. On top of that, if your commits are also
[small](#Small) then the work of understanding _why_ that commit broke things
should leave you investigating a very small number of lines.

As you can see, this quality also entails that your change is testable and
buildable. But what if you realize part way through development that a prior
change (commit) did not behave the way you thought it would, or if you come to
realize you could solve the problem in a more efficient way than you initially
realized. How then would you create a commit that modifies your previous work
AND follows these aforementioned qualities?

## The "Show Your Work" Analogy

If you remember back to any math class, you'll recall that providing the aswer
to a problem, even if you got the right answer, was not enough. You also needed
to _show your work_, meaning a logical progression of changes you need to make
to an equation, one after another, in order to arrive at the correct conclusion.

And as for writing software, "showing your work" naturally leads to the above
principles. Steps are small, they are logical, they are isolated. By the same
analogy, having changes in one commit removed in the next commit would not make
sense in the "show your work" style of commit construction. You would not add 50
to both sides of the equation in one step, and then remove 50 from both sides in
the next step.

Another anology that helps describe the _purpose_ of commits and commit history
(beyond a remote backup of your code) is the idea of giving directions to a
friend to somewhere you just found. You may not have known exactly where to go
at first, maybe you took a turn too late and needed to U-turn. Maybe you took a
wrong turn entirely and needed to retrace your steps to get back on track. It
happens. BUT when you are giving directions to your friend, you wouldn't give
them the EXACT steps you took to get there, you would give them the cleaned-up,
wrong-turns-removed version that gets them straight there.

Now this doesn't mean you need to write perfect code on the first try. You
_will_ take wrong turns; you will need to backtrack and throw away code. So how
do you give your friend (i.e. your commit history) a clean version of what you
wrote?

The answer is... **changing history** 😱

## Opening Pandora's Box

Much like bottom posting and trimming in email[^7], the idea of editing your
commit history seems taboo to many and even unheard of depending on your
background, though it was a very commonly adopted practice early on in each of
these technologies' histories.

In future articles of this git series, we will discuss editing git commit
history as well as when you should make use of this feature and even more
importantly, when you **shouldn't** use this feature.

## Sources

- Much of what is endorsed here comes from Drew DePonte in this
  [article](https://drewdeponte.com/blog/how-we-should-be-using-git/). His
  thoughts are themselves a reproduction of the Linux Kernel development
  standards. Linux in maintained by Linus, the creator of git, who created git
  to manage kernel development.

# Footnotes

[^1]: But also like the atom, this isn't _quite_ true. You can break commits
    apart just like you can break atoms apart into photons, electrons, neutrons,
    etc. However, much like splitting the atom, this is not a task undertaken
    lightly; some caution and experience is required if you don't want to end up
    destroying the universe. We will discuss how to do so in a later post when
    we get to rebases.

[^2]: The imperative mood indicates a command to be followed. Pretend you're
    talking to a robot minion. If you wanted to give a command to your minion,
    you would phrase it, "Cache the api key lookup in the auth controller," not
    "I am caching the api key lookup," or "The api key lookup will be cached,"
    "Can you cache the api key lookup?"

[^3]: <https://en.wikipedia.org/wiki/Single-responsibility_principle>

[^4]: You should definitely fix that bug...just put it in a separate commit with
    a message that describes the bug fix.

[^5]: Picture how you work out a complicated math problem one step at a time by
    transforming your initial problem into a state where you can ultimately
    solve it.

[^6]: But you will perhaps gain a greater appreciation of well constructed
    commits.

[^7]: <https://en.wikipedia.org/wiki/Posting_style>
