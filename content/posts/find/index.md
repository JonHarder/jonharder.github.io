+++
date = '2023-11-03T16:00:00-06:00'
draft = false
title = 'Find'
show_reading_time = true
tags = ['posix', 'how-to']
+++

## Overview

[Find](https://man7.org/linux/man-pages/man1/find.1.html) is a utility to _find_
files on your computer given a wide range of search parameters. It will
recursively walk your filesystem
([breadth-first](https://en.wikipedia.org/wiki/Breadth-first_search)) and print
(by default) all filenames that have matched your search terms. Think of it like
[grep](https://man7.org/linux/man-pages/man1/grep.1.html) but for file names
rather than file contents.

You can search on a variety of metadata on files, ranging from name, creation
time, update time, owner, group, size, and type (file, directory, link, socket,
etc.).

Additionally you can execute a variety of commands on the files `find` finds.

## Terminology

If you read the [docs](https://man7.org/linux/man-pages/man1/find.1.html), which
I [recommend](https://man7.org/linux/man-pages/man1/find.1.html) that you
[do](https://man7.org/linux/man-pages/man1/find.1.html), you'll see referenes to
a handful of terms which may be unfamiliar.

Expression

: The docs state an `expression` is _composed of the "primaries" and
"operands"_. In a less opaque version, it is a search term (with any
accompanying arguments) and something to do to each of the results found.

Primary

: A primary is a `search term` used to limit the list of files returned by find
**OR** an `action to perform` on the list of files returned.

Operand : Operands (or operators as they're sometimes referred as) are ways to
combine expressions, like `or` or `and`.

## Primaries - Finding Files

Enough theory, let's see some examples. For these examples, we will be
investigating a Ruby library meant to query Jira's API.

### By file names

```bash
find . -name '*.yml'
```

```
./.rubocop.yml
./.travis.yml
```

This executes a search starting in the current directory (`.`) for all files
whose name (`-name`) matches the bash glob pattern `*.yml`. The same search can
be done in a case-insensitive fashion using `-iname` instead of `-name`.

Truth be told, this pattern of searching for files by some name glob accounts
for 95% of my use of `find`. But there's so much more it can do.

### By access time

Consider the scenario where you were recently working on a project and wanted to
see which files you have edited in the last 30 minutes.

```bash
find . -amin -30 -name '*.rb'
```

```
./lib/jira.rb
```

The `-amin` primary finds all files that have been accessed 30 or fewer minutes
ago. We specified `-30` and not `30` to represent "30 or fewer". If we wanted to
only find files that have been accessed _more_ than 30 minutes ago, we could
specify the primary with a `+` (`-amin +30`). This use of `+` and `-` applies to
any primary that takes a numeric argument.

If you wanted to look at the files changed over a longer period of time without
having to pull in a
[command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)
(`-amin -$(expr 60 \* 24 \* 15)`), you can instead use `-atime` which excepts a
number and a time specifier. `s` for seconds, `m` for minutes, `h` for hours,
`d` for days, and `w` for weeks.

### By creation time

If you were instead interested in the file _creation_ time rather than _access_
time, you could use `-Bmin` and its counterpart `-Btime` which peform the same
searches as `-amin` and `-atime` but for file creation time.

```bash
find . -Btime -3d
```

```
./lib/jira/api/just_created.rb
```

### By owner and group

In some instances it can be handy to find files owned by a particular user, or
particular group. Find has you covered.

```bash
find . -user root
```

```
./bin/stray_file
```

This finds all the files owned by `root`. Similaraly, `-group GROUP` finds all
the files set to the group `GROUP`.

I rarely use these flags as I rarely work on machines with multiple users or
exotic group membership, but if you're a system administrator, these could be
very handy. Regardless, learning more about the UNIX file permission model is
very helpful so I would recommend reading up more on it if you're curious.[^1]

### By type and size

Shifting gears, let's say you're trying to track down files that are hogging
your hard drive space. `-size` helps you find files larger or smaller than your
given size.

```bash
find . -size +512k
```

```
./coverage/index.html
```

This will find all files larger than 512 kilobytes. You can also search in units
of megabytes with `M`, gigabytes with `G`, terabytes with `T` and if you're
working with truly titanic files, petabytes using `P`.

I need to come clean about something. I've used the term `files` throughout this
article to refer to the results that find returns from its searches, but that's
not exactly true. If you've experimented with find yourself, you have even
noticed that find doesn't _just_ return files. It returns everything on the
filesystem: files, directories, sybolic links, sockets, and more.

You can tell `find` to only return results of a given type with the `-type`
primary.

```bash
find . -type d ! -path '*git*' ! -path '*coverage*'
```

```
.
./bin
./spec
./spec/issue
./spec/api
./spec/report
./.yardoc
./.yardoc/objects
./lib
./lib/jira
./lib/jira/issue
./lib/jira/api
./lib/jira/report
./doc
./doc/css
./doc/js
./doc/Jira
./doc/Jira/Reporting
./doc/Jira/Issue
./doc/Jira/Api
./doc/Jira/Report
./doc/Jira/Client
./.idea
```

This returns only the directories in the current directory and its children. The
`-path` primary searches in the whole path of the file, not just the file
itself, and the `!` will be covered later when we talk about
[operands](#Operands)

## Primaries - Acting on files

So far `find` has only been printing the results that it finds, but you can take
other actions on the matched files as well. This is implicitly the same as using
the `-print` primary. Similar to it is `-ls`, which formats its output as though
you had ran `ls -l` on the list of files returned.

```bash
find . -type f -name '*.md' -ls
```

```
30288144        8 -rw-r--r--    1 jharder          staff                 622 Feb  8  2023 ./CHANGELOG.md
35271368       24 -rw-r--r--    1 jharder          staff               11914 Apr 13  2023 ./README.md
```

If instead you wanted to clean up a bunch of files, you can use find with the
`-delete` primary to delete all the files find finds. No reaching for
[xargs](https://man7.org/linux/man-pages/man1/xargs.1.html) necessary.

You can run any utility you want on the list of files using the `-exec` primary,
though there are two important things to keep in mind:

1. You **must** terminate the command with an escaped `;` in order to tell find
   when your exec command ends and the next primary begins.
2. To reference the filename in the utility us `{}`.

```bash
find . -type f -name '*.md' -exec echo {} \;
```

```
./CHANGELOG.md
./README.md
```

You can replicate the `-delete` primary by using `-exec rm {} \;` but I might
wonder why you're doing that. A helpful variation of `-exec` is `-ok`, which is
identical to `-exec` but asks you for permission first. You you wanted to delete
a handful of files but wanted to confirm before they get deleted `-ok` is a
great option.

```bash
find . -empty -ok rm {} \;
```

```
"rm ./lib/empty_file_2"? no
"rm ./lib/jira/api/just_created.rb"? no
"rm ./lib/empty_file_1"? yes
```

## Operands

Operands are quite simple, in fact, most of these examples have been using them!
The operators listed in the
[man](https://man7.org/linux/man-pages/man1/man.1.html) pages list `(` `)`, `!`,
`-and`, and `-or`. When multiple primaries are provided to find, `-and` is
implicitly used to link them together. `find . -empty -atime +1w` is the same as
typing `find . -empty -and -atime +1w`, meaning, find all the files that are
empty **AND** haven't been accessed in at least a week.

`( expression )` sets an order of precedence similar to how paretheses work in
math. `(1+3) * 4` would evaluate `1+3` before `* 4` because of the parentheses.
`-or` and `!` (also expressed as `-not`) are similar, and follow boolean logic
that you should be familiar with.

# Conclusion

Find is a humble utility that seems simple on its face, but weilds great power
in the hands of those who know how to handle it. I've used find for years mostly
to locate files with a certain name or file extension, but in researching this
article I've found numerous other use cases.

I encourage you to poke around and consider find when you're looking for some
files and don't know where they're stored. Or you could use it to find files
bigger than `1G` and consider cleaning them up. Or finding empty files that have
been sitting around for a long time.

# Footnotes

[^1]: <https://mason.gmu.edu/~montecin/UNIXpermiss.htm>
