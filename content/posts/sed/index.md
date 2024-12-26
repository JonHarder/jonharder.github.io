+++
date = '2023-11-10T16:00:00-06:00'
draft = false
title = 'Sed (Part 1)'
tags = ['posix', 'how-to']
+++

## Overview

[Sed](https://man7.org/linux/man-pages/man1/sed.1.html) stands for (s)tream
(ed)itor and is useful for taking any input you have and modifying it in some
way before sending it along as output, either to the console or as input to the
next command in your pipeline. It operates on each line of your input, executing
some command on each as the input is read.

Chances are if you've encountered `sed` before, it came from some sage linux
master on stack overflow when providing an answer to a bash scripting question.
If you've used `sed` yourself for some task, it's likely using the `substitute`
command to find and replace some search term.

You may have seen a command like this before:

```bash
echo "this is line one\nthis is line two" | sed 's/line/snake/'
```

```
this is snake one
this is snake two
```

However, `sed` can do so much more, if only you spend the time to figure out
what on earth it's talking about in its manual. In general, a command is defined
in the man page as follows:

> \[address\[,address\]\]function\[arguments\]

But to unpack this fully, we will need to clear up some strange terminology.

## Terminology

Address

: If all you've ever used `sed` to do is substitute terms using
`s/term/replacement/`, the concept of an `address` might not make sense at
first. If one is provided, it will be before the function (`s` - substitute, in
this case). An `address` provides some way to restrict which lines of the input
`sed` will actually operate on. You can also optionally provide a second address
to create a range of lines for `sed` to restrict its operation over.

    An address can be one of three things:

    1.  a number representing which line(s) of input to act on. ex
        `3,7s/apple/banana/`
    2.  the `$` sign, meaning the last line of input
    3.  a regex, meaning only lines of input which match the regex will
        be passed along to the function

    In this example, we\'re telling `sed` to execute the
    substitude command only on lines 2 through four. For lines within
    that range, the provided function will be ran. For lines **outside**
    the range, lines will be left untouched and printed as is.

    ``` 
    sed '2,4s/line/fish/'
    ```

    In the next example, `$` is used as the terminating address to
    restrict the lines `s` operates on.

    ```
    sed '3,$s/foo/bar/'
    ```

    In this example, there are two lines with the word `two`.
    If we just did a substitute command without an address, **all**
    lines with the word `two` would be changed to
    `snake`. But because we are providing a regex address
    (restricting to lines containing `apple`), only line four
    (containing the word `apple`) actually has the text in
    the substitute command replaced.

    ``` bash
    echo "line one\nline two\nline two\napple two" | sed '/apple/s/two/snake/'
    ```

    ```
    line one
    line two
    line two
    apple snake
    ```

function

: The actual thing you want `sed` to **do**.

arguments

: Some functions take arguments. We'll cover this a bit more when covering some
of those functions.

cycle

: Because `sed` is line oriented, it filters input and executes functions on a
per line bases. Each round of reading in a line from input, checking if the line
is within the given address, executing the function, and printing output
constitutes one _cycle_.

## Functions

So far we're only shown examples using the (s)ubstitute function. There are many
more. Let's start with a simple one. The `p` function "_Writes the pattern space
to standard output._". Ok, hold up. What's a _pattern space_? To answer that,
we'll need to dig a bit deeper into how `sed` works.

And to do that, let's visualize a some of the moving parts of a `sed` execution
with the following input:

```bash
echo "This is the first line of text.
And this is another line.
Here's another for you.
This is anonther line with the word 'line' in it.
and this is the last line." > example.txt
```

Say we have the given `sed` command: `sed '2,4s/line/REPLACED/'`. The addresses
are the lines 2 through four. The command is `substitute` the text `line` with
`REPLACED`.

| line no. | pattern space                                    | output                                               |
| :------- | :----------------------------------------------- | :--------------------------------------------------- |
| 1        | This is the first line of text.                  | This is the first line of text.                      |
| 2        | And this is another line.                        | And this is another REPLACED.                        |
| 3        | Here's another for you.                          | Here's another for you.                              |
| 4        | This is another line with the word 'line' in it. | This is another REPLACED with the word 'line' in it. |
| 5        | and this is the last line.                       | and this is the last line.                           |

Let's walk through each line `sed` read and operated on (remember this is called
a `cycle`). Line one loaded the text into the pattern space, which is a
temporary buffer used to operate on with the given function. `Sed` loads each
line into the pattern space regardless of if the line falls within the address
(if provided).

Next `sed` checks if the line _is_ within the provided address (`2,4` in this
case). Because this is line 1, it does **not** fall within the address provided,
so `sed` just prints it to output without operating on it.

After this, it clears the pattern space, and a new cycle begins by loading the
next line of input into the pattern space.

Line 2 _is_ within the pattern provided, so the substitute function will run. We
provided the arguments `line` and `REPLACED` to the function, and so the output
reflects that substitution.

The same happens for line 3, except there was no match for the substitute to
replace. Line 4 is the last line in the address space of `2,4`, so `line` is
replaced, however you will notice only the **first** occurrence of `line` was
replaced. By default, `sed` will only find the first instance. This can be
changed if you provide the `g` flag to the function: `sed 's/foo/bar/g'` . Line
5 is outside the address space, and so `line` is **not** replaced.

```bash
sed '2,4s/line/REPLACED/' example.txt
```

```
This is the first line of text.
And this is another REPLACED.
Here's another for you.
This is anonther REPLACED with the word 'line' in it.
and this is the last line.
```

Additionally, the behavior of, _print the line, unchanged, when outside the
address_ can be modified by using the `-n` flag.

With all of this, we're now ready to look at the `p` function again. Consider
our example again.

```bash
sed '2,4s/line/REPLACED/p' example.txt
```

```
This is the first line of text.
And this is another REPLACED.
And this is another REPLACED.
Here's another for you.
This is anonther REPLACED with the word 'line' in it.
This is anonther REPLACED with the word 'line' in it.
and this is the last line.
```

Whith the additional `p` function executing after the `s` function, only the
lines that `s` successfully operated on get passed along to the `p` function.
This is why line 3 was not printed twice even though it was within the address
range.

Now let's try it without printing by default.

```bash
sed -n '2,4s/line/REPLACED/p' example.txt
```

```
And this is another REPLACED.
This is anonther REPLACED with the word 'line' in it.
```

Is this what you expected? Hopefully by this point it should make sense. Lines
are not printed by default because of the `-n` flag, and only lines that passed
`s` successfully were then sent to `p` to be printed.

# Next time

The more I dug into said, the more powerful I discovered it really was. This is
only the beginning. Hopefully this alone was helpful to try and understand the
[man](https://man7.org/linux/man-pages/man1/sed.1.html) page on `sed`.

Next time will be almost entirely examples of using all of the other functions
`sed` provides and fun ways to combine them into surprisingly powerful commands.
