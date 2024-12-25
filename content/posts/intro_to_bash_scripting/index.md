+++
date = '2023-12-22T16:00:00-06:00'
draft = false
title = 'Intro to Bash Scripting'
show_reading_time = true
tags = ['posix', 'how-to']
+++


# Overview

Bash is a (perhaps *the* ubiquitous) shell. As a member of the POSIX
interface, bash provides a consistent access point to applications on
your operating system.

Because it is all but guaranteed to be installed on any server and
environment, learning this one interface will ensure you can be
comfortable interacting with any computer.

Before we begin, it will be helpful to clear up a few ambiguous terms
related to command line tools that often come up together. A *terminal*
is a desktop application you start which presents you with a *command
line* interface, running your chosen *shell*.

The terminal is the user interface to interact with your computers
files, devices, and hardware.

The command line is a method of interacting with your computer where you
input a command, the computer processes the command, computes the
result, and prints it out.

The shell is the software that is responsible for the actual processing
of your requests and returning the results to the user.

So in this case, we are interested in a *shell*, [bash](https://www.man7.org/linux/man-pages/man1/bash.1.html). Bash
was written as a open source replacement to the
[bourne](https://en.wikipedia.org/wiki/Bourne_shell) shell. First
released in 1989, it's been a staple for decades, only recently being
overtaken by [zsh](https://en.wikipedia.org/wiki/Z_shell) as the default
shell on Mac OS among other systems.

Bash can be used to simply execute programs installed on your computer:

``` bash
ls ~/Code/k8s-app
```

```
backend
docker-compose.yml
files
frontend
kubernetes
testing.yml
```

But it can do much more. Bash introduced control flow mechanisms
allowing you to create reusable scripts along the lines of something you
would expect from a python script.

# Control flow

## if, else, etc.

Like any scriptable, programable environment, bash supports branching
features depending on provided conditions.

```bash
if true
then
  echo truthy
fi
```

```
truthy
```

If checks (perhaps confusingly if you're coming from a software
development backgroud) if the condition evaluates to 0. If it does, the
`then` branch is executed. Also note an `if` block is terminated with
`fi` (if, backwards). Non zero results in if following the
`else` branch if one is given.

```bash
if false
then
  echo truthy
else
  echo falesy
fi
```

```
falesy
```

If checks the [exit status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)
of it's condition in order to determine which branch to take. the
[true](https://man7.org/linux/man-pages/man1/true.1.html) command simply sets its exit status to 0 (truthy).
[false](https://man7.org/linux/man-pages/man1/false.1.html) does the inverse. You can check the exit status of
the previous command with the variable `$?`.

```bash
true
echo $?

false
echo $?
```

```
0
1
```

Every command ran in the shell sets an exit command, so you can use this
to branch based on the success or failure of any command.

```bash
if grep "foo" ~/Code/k8s-app/backend/main.go
then
  echo "found foo"
else
  echo "couldn't find foo"
fi
```

```
couldn't find foo
```

In the above example, we can tell if the [grep](/posts/grep)
command failed or not, but if we look at another example where grep does
succeed, the results aren't probably what you're expecting.

```bash
if grep "http" ~/Code/k8s-app/backend/main.go
then
  echo "Found it!"
else
  echo "Couldn't find it."
fi
```

```
    "net/http"
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
func indexHandler(app *App) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
func recipeHandler(app *App) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
    handler func(app *App) http.HandlerFunc
func initServer(app *App) *http.ServeMux {
    mux := http.NewServeMux()
        otelHandler := otelhttp.NewHandler(http.HandlerFunc(route.handler(app)), route.route)
    log.Fatal(http.ListenAndServe(":8080", mux))
Found it!
```

In this example, grep succeeded and because grep prints the lines that
match its search term, those results showed up in the output as well.
Sometimes this could be what you want, but often it's not.

What we need to do here is to test the exit status only, but not print
anything that the command itself would otherwise output. This can be
accomplished by redirecting grep's output to a black hole,
[/dev/null](https://www.digitalocean.com/community/tutorials/dev-null-in-linux).

## while, for

Bash also supports while and for loops. Notice that `for` and
`while` use the keyword `do` before the loop body,
and `done` to close it.

```bash
for x in 1 2 3 4
do
  echo $x
done
```

```
1
2
3
4
```

``` bash
for file in ~/Code/k8s-app/backend/*.go
do
  echo $file
done
```

```
/Users/jharder/Code/k8s-app/backend/app.go
/Users/jharder/Code/k8s-app/backend/db.go
/Users/jharder/Code/k8s-app/backend/main.go
/Users/jharder/Code/k8s-app/backend/tracing.go
/Users/jharder/Code/k8s-app/backend/tracing_test.go
```

Some of this wont make perfect sense yet until we cover
[variables](#variables) and [shell arithmetic](#shell-arithmetic), but hopefully
you can get a sense of what it's doing:

``` bash
x=5
while [ $x -gt 0 ]
do
  echo $x
  x=$(( $x - 1 ))
done
```

```
5
4
3
2
1
```

## Case statements

Bash also supports case (also referred to as `switch`
statements. The syntax may look a little foreign to our 21st century
eyes, but it is handles the key concepts of most switch statements.

``` bash
x=5
case $x in
1)
  echo "$x = 1"
  ;;
2 | 3 | 4 | 5)
  echo "$x is between 2 and 5"
  ;;
*)
  echo "$x is something else"
  ;;
esac

```

```
5 is between 2 and 5
```

Each arm of the case statement can be a raw value, or a pattern. Cases
patterns end with `)`, and multiple patterns can be provided
separated with `|`. Case blocks end with `;;`.
Lastly like `if`, case statements end with their keyword
reversed, `esac`.

# Output redirection

The output of a command can be redirected (from standard output) to
another location. This could be a different stream (like standard out),
a file, or a virtual device. `/dev/null` gobbles up all data
sent to it so in this case the results grep prints are never shown.

``` bash
if grep "http" ~/Code/k8s-app/backend/main.go > /dev/null
then
  echo "found it"
else
  echo "couldn't find it"
fi
```

```
found it
```

Sometimes there is no command which exists to check the condition you
want. In these scenarios, you can use the command
[test](https://www.man7.org/linux/man-pages/man1/test.1.html).

# test

The man page for test is very informative. It supports a number of flags
which execute a number of different conditional checks. For integers it
supports `-eq`, `-ne`, `-gt`,
`-ge`, `-lt`, and `-le`, or
`==`, `!=`, `>`, `>=`,
`<`, and `<=` expressed in a more familiar syntax.

``` bash
if test 4 -gt 3
then
  echo "Four is greater than three"
else
  echo "Four is NOT greater than three"
fi
```

```
Four is greater than three
```

To make things a little nicer to look at, bash provides an alias of
`test` called `[`.[^1]

``` bash
if [ 4 -gt 3 ]
then
  echo "Four is greater than three"
else
  echo "Four is NOT greater than three"
fi
```

```
Four is greater than three
```

*NOTE*: because `[` is a bash command just like `test` or `true`, you
**must** have a space after it. It\'s tempting to say `if [condition]`
or even `if[condition]`, but since `[` is just an alias of `test` which
is a regular command, this is illegal as it would be equivelant to
`if testcondition` or `iftestcondition` respectively.

You can test for different file system conditions using test as well.

``` bash
if [ -e ~/Code/k8s-app/backend/main.go ]
then
  echo "file exists!"
else
  echo "File not found"
fi
```

```
file exists!
```

``` bash
if [ -d ~/Code/k8s-app/backend/ ]
then
  echo "file exists, and it's a directory"
else
  echo "either the file doesn't exist, or it does, but it's not a directory"
fi
```

```
file exists, and it's a directory
```

``` bash
if [ -r ~/Code/k8s-app/backend/main.go ]
then
  echo "file exists and it has read permissions"
else
  echo "file may or may not exists, but if it does, it can't be read"
fi
```

```
file exists and it has read permissions
```

# Variables

## Simple variables

Bash also supports variable, which are set using `var=value`
syntax. To set a variable you do not provide the `$` symbol
in front of the variable, but to retrieve it, you must reference the
variable with the `$` symbol.

``` bash
my_var=5
echo $my_var
```

``` example
5
```

Variables can be set to the result of commands if you use a sub-shell to
compute the result.

``` bash
my_var=$(ls -l ~/Code/k8s-app/backend | tail -n1)
echo $my_var
```

```
-rw-r--r-- 1 jharder staff 65 Dec 15 13:03 tracing_test.go
```

You can also embed variables inside of a string to perform string
interpolation.

``` bash
my_name="Jon"
echo "Hello, $my_name"
```

```
Hello, Jon
```

If you want to put content directly after your variable, you'll need to
use `${...}` instead of `$`.

``` bash
animal=cat
echo "I like ${animal}s"
```

```
I like cats
```

If we want to store arithmetic computations in a variable we will need
to reach for some additional constructs because sadly, bash will not
perform arithmetic operations by itself.

``` bash
a=4+8
echo $a
```

```
4+8
```

To do arithmetic in the shell, you'll need to use....
[shell arithmetic](https://www.gnu.org/software/bash/manual/html_node/Shell-Arithmetic.html).

## Arrays

Bash also supports arrays.

```bash
numbers=(5 2 6 7)
echo $numbers
```

```
5
```

Is that what you expected? It sure wasn't what I expected. Referencing
the variable just gives you the first element in the array, similar to
C.

You can access elements of the array using `${var[n]}`
syntax. You can access all the elements by using `@` as the
array index.

``` bash
numbers=(5 2 6 7)
echo ${numbers[2]}
echo ${numbers[@]}
```

```
6
5 2 6 7
```

The latter can be used in a for loop to handle every element.

``` bash
my_array=(thing1 thing2 "thing three")
for x in "${my_array[@]}"; do
  echo "x = $x"
done
```

```
x = thing1
x = thing2
x = thing three
```

The double quoting around `${my_array}` is to avoid splitting
on the space in `"thing three"`. Try removing the quotes and
see what happens.

# Shell arithmetic

```bash
a=$((4+8))
b=$((4<3))
c=$((4>3))

echo $a
echo $b
echo $c
```

```
12
0
1
```

You can reference variables inside `(( ))` without dollar
signs if you want.

```bash
x=6

echo $((x*8))
echo $((4>3))
```

```
48
1
```

You can combine this with `if` for more concise conditions...

``` bash
if $((4>3))
then
  echo "four is greater than three"
else
  echo "four is not greater than three"
fi
```

```
four is not greater than three
```

however, because shell arithmetic returns the more familiar
`1` for true and `0` for false, you'll need to
use `test` (or `[`) to mediate between math and
your shell.

``` bash
if [ $((4>3)) ]
then
 echo "four is greater than three"
else
 echo "four is not greater than three"
fi
```

```
four is greater than three
```

# Functions

## Definition

Bash also has basic support for defining and using functions, with some
oddities.

``` bash
function my_fun() {
  echo "hi"
}

my_fun
```

```
hi
```

## Defining arguments

You would think that adding arguments inside the `()` would
allow you to define parameters to your function. And you'd be wrong.

``` bash
# THIS DOES NOT COMPILE!!!
function my_fun(a, b) {
  echo "$a $b"
}
```

Instead, you need to refer to the arguments by their positional index:
`$1`, `$2`, ..., `$n`.

``` bash
function my_fun() {
  echo "arg 1 = $1"
  echo "arg 2 = $2"
}

my_fun 1 2
```

```
arg 1 = 1
arg 2 = 2
```

## Local variables

You can make local variables to the function without polluting the
global namespace using the keyword `local`.

``` bash
x=3
echo "before my_fun, x=$x"

function my_fun() {
  local x=5
  echo "in my_fun, x=$x"
}

my_fun

echo "after my_fun, x=$x"
```

```
before my_fun, x=3
in my_fun, x=5
after my_fun, x=3
```

## Returning values

Because functions operate just like any other command you may run in the
shell, you can only set an exit status, not any arbitrary value. To
return anything else, you can use standard output and capture that value
in a subshell.

``` bash
function my_fun() {
  echo "$1 times $2 is $(($1 * $2))"
}

result=$(my_fun 3 6)
echo $result
```

```
3 times 6 is 18
```

# Conclusion

Bash as a bit of a quirky programming language, but being a programming
language that's built right into your shell can be very handy. However,
because it lacks any higher level language constructs like classes,
higher order functions, or clear argument or type definitions, bash
scripts have a maximum carrying capacity in the form of maintainability.
When they get too large, it's probably best to look for a dedicated
programming language.

# Footnotes

[^1]: Bash also adds `[[` as an extension on top of the posix `[`. It
    functions in much the same way but adds some niceties on top of it.
    Space does not allow me to go into detail, but starting
    [here](https://superuser.com/a/1533931) provides a succinct overview
    of the differences.
