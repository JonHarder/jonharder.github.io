+++
date = '2023-10-27T16:00:00-06:00'
draft = false
title = 'Grep'
tags = ['posix', 'how-to']
+++

## Overview

Grep is a fundamental part of the command line toolbelt. True to the
[Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy), grep does one
thing, and does it well. Its job is to search input for a given search pattern
and print all lines that match. Think of it like `Command-f` for the
terminal.

Its origins date back to the `ed` text editor. It was a common practice within
`ed` to execute the command `g/r(egular) e(expression)/p`, meaning,
`globally` search for lines that match the given `regular expression` and
`print` them. This functionality was so useful, that the name \"grep\" became
synonomous with the act of searching files and printing matches, and a
standalone `grep` program was born.[^1]

There are two ways to have `grep` search text. One is to provide a list of files
as the target of the search which will be searched line-by-line for your pattern
and matching lines will be printed. The other is to search incoming text from
[stdin](man:stdin) and print the matching lines.

## Examples - Searching files

### Basics

Let's look at some examples, looking in the same `k8s-app` project you've seen
in all previous examples. In this case, we'll focus first in the `backend`
directory.

```bash
echo "Current directory: $(pwd)"
echo "----------"
ls
```

```
Current directory: /Users/jharder/Code/k8s-app/backend
----------
Dockerfile
app.go
db.go
go.mod
go.sum
main.go
tracing.go
```

Let's say you're interesting in finding all occurrences of the phrase `trace`.

```.bash
grep trace *
```

```
app.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
app.go:func newApp(ctx context.Context, tp *sdktrace.TracerProvider) (*App, error) {
db.go:    sdktrace "go.opentelemetry.io/otel/sdk/trace"
db.go:func dbConn(ctx context.Context, tp *sdktrace.TracerProvider) (*mongo.Client, error) {
go.mod:   go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0
go.mod:   go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0
go.mod:   go.opentelemetry.io/otel/trace v1.14.0
go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0 h1:TKf2uAs2ueguzLaxOCBXNpHxfO/aC7PAdDsSH0IbeRQ=
go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0/go.mod h1:HrbCVv40OOLTABmOn1ZWty6CHXkU8DK/Urc43tHug70=
go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0 h1:3jAYbRHQAqzLjd9I4tzxwJ8Pk/N6AqBcF6m1ZHrxG94=
go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0/go.mod h1:+N7zNjIJv4K+DeX67XXET0P+eIciESgaFDBqh+ZJFS4=
go.sum:go.opentelemetry.io/otel/trace v1.14.0 h1:wp2Mmvj41tDsyAJXiWDWpfNsOiIyd38fy85pyKcFq/M=
go.sum:go.opentelemetry.io/otel/trace v1.14.0/go.mod h1:8avnQLK+CG77yNLUae4ea2JDQ6iT+gozhnZjy/rw9G8=
main.go:  "go.opentelemetry.io/otel/trace"
main.go:      span := trace.SpanFromContext(ctx)
main.go:          log.Printf("Error shutting down tracer provider: %v", err)
tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace"
tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp"
tracing.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
tracing.go:func initExporter(ctx context.Context) (*otlptrace.Exporter, error) {
tracing.go:   client := otlptracehttp.NewClient()
tracing.go:   exporter, err := otlptrace.New(ctx, client)
tracing.go:func initTracer(ctx context.Context) (*sdktrace.TracerProvider, error) {
tracing.go:   // https://github.com/open-telemetry/opentelemetry-go/blob/main/exporters/otlp/otlptrace/otlptracehttp/example_test.go#L68
tracing.go:   // exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
tracing.go:   trace_provider := sdktrace.NewTracerProvider(
tracing.go:       sdktrace.WithSampler(sdktrace.AlwaysSample()),
tracing.go:       sdktrace.WithBatcher(exporter),
tracing.go:       sdktrace.WithResource(resource.NewWithAttributes(semconv.SchemaURL, semconv.ServiceName(name))),
tracing.go:   otel.SetTracerProvider(trace_provider)
tracing.go:   return trace_provider, nil
```

That's a lot of results, but we probably don't care about `go.mod` and `go.sum`.
We can trim the results down by specifying a
[glob](https://tldp.org/LDP/abs/html/globbingref.html) that feeds only specific
files to grep to search through.[^2] Let's search for only Go files (with the
`.go` suffix).

If we instead run `grep trace **.go`, our shell will expand `*.go` to
`app.go db.go main.go tracing.go`[^3], and then feed that as arguments to
`grep`. Let's see the results:[^4]

```bash
grep trace *.go
```

```
app.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
app.go:func newApp(ctx context.Context, tp *sdktrace.TracerProvider) (*App, error) {
db.go:    sdktrace "go.opentelemetry.io/otel/sdk/trace"
db.go:func dbConn(ctx context.Context, tp *sdktrace.TracerProvider) (*mongo.Client, error) {
main.go:  "go.opentelemetry.io/otel/trace"
main.go:      span := trace.SpanFromContext(ctx)
main.go:          log.Printf("Error shutting down tracer provider: %v", err)
tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace"
tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp"
tracing.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
tracing.go:func initExporter(ctx context.Context) (*otlptrace.Exporter, error) {
tracing.go:   client := otlptracehttp.NewClient()
tracing.go:   exporter, err := otlptrace.New(ctx, client)
tracing.go:func initTracer(ctx context.Context) (*sdktrace.TracerProvider, error) {
tracing.go:   // https://github.com/open-telemetry/opentelemetry-go/blob/main/exporters/otlp/otlptrace/otlptracehttp/example_test.go#L68
tracing.go:   // exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
tracing.go:   trace_provider := sdktrace.NewTracerProvider(
tracing.go:       sdktrace.WithSampler(sdktrace.AlwaysSample()),
tracing.go:       sdktrace.WithBatcher(exporter),
tracing.go:       sdktrace.WithResource(resource.NewWithAttributes(semconv.SchemaURL, semconv.ServiceName(name))),
tracing.go:   otel.SetTracerProvider(trace_provider)
tracing.go:   return trace_provider, nil
```

### Excluding files

It's a bit annoying to have to deal with `tracing.go` polluting our results as
it's obviously going to have a bunch of hits which we don't care about. There's
no quick and easy way to exclude it from the shell glob, but `grep` itself gives
us a way to exclude certain files from being processed when it runs. `--exclude`
takes a pattern that it will run against the name of each file it will process,
if the pattern matches, `grep` will skip that file. The opposite behavior can be
achieved with `--include`.

```bash
grep --exclude 'tracing.go' trace *.go
```

```
app.go: sdktrace "go.opentelemetry.io/otel/sdk/trace"
app.go:func newApp(ctx context.Context, tp *sdktrace.TracerProvider) (*App, error) {
db.go:  sdktrace "go.opentelemetry.io/otel/sdk/trace"
db.go:func dbConn(ctx context.Context, tp *sdktrace.TracerProvider) (*mongo.Client, error) {
main.go:    "go.opentelemetry.io/otel/trace"
main.go:        span := trace.SpanFromContext(ctx)
main.go:            log.Printf("Error shutting down tracer provider: %v", err)
```

### Line numbers

Having these results is handy, but it would be better to know where in the file
these matches can be found. Passing the `-n` or `--line-number` flag provides
the line number of each hit.

```bash
grep -n trace
```

```
11: "go.opentelemetry.io/otel/trace"
26:     span := trace.SpanFromContext(ctx)
77:         log.Printf("Error shutting down tracer provider: %v", err)
```

### Context

This is indeed nicer, but it would be nicer _still_ if we had a bit of context
for each of the matches. What is going on around the lines where these results
were found?

You can use the `-C` or `--context` argument with a positive number and that
number of lines before and after the match will be printed to stdout as well.

```bash
grep -n --context=3 trace main.go
```

```
8-
9-    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
10-   "go.opentelemetry.io/otel"
11:   "go.opentelemetry.io/otel/trace"
12-)
13-
14-const name = "recipe-service"
--
23-   return func(w http.ResponseWriter, r *http.Request) {
24-       ctx := r.Context()
25-       // grabs the current span
26:       span := trace.SpanFromContext(ctx)
27-       span.AddEvent("GetRecipes")
28-       log.Println("GET /recipes")
29-       recipes, err := app.getRecipes(ctx)
--
74-   }
75-   defer func() {
76-       if err := tp.Shutdown(context.Background()); err != nil {
77:           log.Printf("Error shutting down tracer provider: %v", err)
78-       }
79-   }()
80-
```

Lines ending in `-` signify they are lines added for context, lines ending in
`:` signify the matched line itself, and `--` separates each matched line (and
its context).

You can also specify the number of lines to display before and after separately
using `B` or `--before-context` and `-A` or `--after-context` respectively.

### Count

Sometimes it can be helpful to understand how often a particular search term
appears in your searched files. For this, you can use `-c` or `--count`.

```bash
grep -c trace *.go
```

```
app.go:2
db.go:2
main.go:3
tracing.go:15
```

### Matching files only

If you didn't care about _which_ lines specifically mached your pattern, but
only which _files_ contained the match, you can use the `-l` or
`--files-with-matches` flag to only show the files instead.

```bash
grep -l --exclude '*.sum' --exclude '*.mod' trace *
```

```
app.go
db.go
main.go
tracing.go
```

You can also see here that the exclude (and their `--include` couterpart)
actually take a shell glob as their argument, not just an exact file name. Here
we use this to exclude _all_ files ending in `.sum` or `.mod`.

### Printing mached content only

Perhaps the inverse of showing the files that match is showing the match only,
not the rest of the line. This is accomplished using the `-o` or
`--only-matching` flag. This is likely most handy when paired witn the `-E` or
`--extended-regexp` flag which enables full regular expression support.

The regular expression searches for anything that starts with `With` and matches
any number of characters that aren\'t the `(` symbol. This has the effect of
finding any function definition or call starting with `With`. Using `-o` means
all we get back is what actually mached the regular expression, and which file
it came from. If we were interested in knowing where in the file the match was,
we could add the `-n` flag as well.

```bash
grep --exclude '*.sum' --exclude '*.mod' -o -E 'With[^(]*' *
```

```
db.go:WithTracerProvider
tracing.go:WithPrettyPrint
tracing.go:WithSampler
tracing.go:WithBatcher
tracing.go:WithResource
tracing.go:WithAttributes
```

### Searching recursively

Now suppose you didn't know which folder contained the files you were looking
for. Instead you can run `grep` from any directory and pass the `-r` or
`--recursive` flag to change grep from searching through only the files
explicitly given to searching all files recursively in the provided directory
and all children directories and files.

Lets go up one directory from our go app example. In the parent directory we
have directories for multiple applications and the IaC kubernetes code to deploy
each of them:

```bash
echo "Current Directry: $(pwd)"
echo "---------------"
ls -F
```

```
Current Directry: /Users/jharder/Code/k8s-app
---------------
backend/
docker-compose.yml
files/
frontend/
kubernetes/
```

We may not want to restrict our search to just the backend; let's search for
`trace` from here using the recursive flag.

When searching recursively, you can specify the directry to start the search
from in place of the files to search in. If you don\'t provide a directory, the
current working directory will be used instead

```bash
grep -r trace | head -n 10
```

```
./frontend/node_modules/@types/express-serve-static-core/index.d.ts:    trace: IRouterMatcher<this>;
./frontend/node_modules/@types/express-serve-static-core/index.d.ts:    trace: IRouterHandler<this, Route>;
./frontend/node_modules/@types/node/globals.d.ts:     * Optional override for formatting stack traces
./frontend/node_modules/@types/node/globals.d.ts:     * @see https://v8.dev/docs/stack-trace-api#customizing-stack-traces
./frontend/node_modules/@types/node/ts4.8/globals.d.ts:     * Optional override for formatting stack traces
./frontend/node_modules/@types/node/ts4.8/globals.d.ts:     * @see https://v8.dev/docs/stack-trace-api#customizing-stack-traces
./frontend/node_modules/@types/node/ts4.8/tls.d.ts:         * When enabled, TLS packet trace information is written to `stderr`. This can be
./frontend/node_modules/@types/node/ts4.8/tls.d.ts:         * Note: The format of the output is identical to the output of `openssl s_client -trace` or `openssl s_server -trace`. While it is produced by OpenSSL's`SSL_trace()` function, the format is
./frontend/node_modules/@types/node/ts4.8/tls.d.ts:         * When enabled, TLS packet trace information is written to `stderr`. This can be
./frontend/node_modules/@types/node/ts4.8/trace_events.d.ts: * The `trace_events` module provides a mechanism to centralize tracing information
```

I'll save you from scrolling through the thousands of lines of results from
`./frontend/node_modules/` by just displaying the first 10 lines using
[head](https://man7.org/linux/man-pages/man1/head.1.html).

Grep can save us from this annoyance as well; when searching recursively, you
can exclude whole directories from its search in the same way that you could
exclude individual files. Rather than using `--exclude`, we can use
`--exclude-dir`

```bash
grep -r --exclude-dir 'node_modules' trace
```

```
./backend/go.mod:   go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0
./backend/go.mod:   go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0
./backend/go.mod:   go.opentelemetry.io/otel/trace v1.14.0
./backend/db.go:    sdktrace "go.opentelemetry.io/otel/sdk/trace"
./backend/db.go:func dbConn(ctx context.Context, tp *sdktrace.TracerProvider) (*mongo.Client, error) {
./backend/tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace"
./backend/tracing.go:   "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp"
./backend/tracing.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
./backend/tracing.go:func initExporter(ctx context.Context) (*otlptrace.Exporter, error) {
./backend/tracing.go:   client := otlptracehttp.NewClient()
./backend/tracing.go:   exporter, err := otlptrace.New(ctx, client)
./backend/tracing.go:func initTracer(ctx context.Context) (*sdktrace.TracerProvider, error) {
./backend/tracing.go:   // https://github.com/open-telemetry/opentelemetry-go/blob/main/exporters/otlp/otlptrace/otlptracehttp/example_test.go#L68
./backend/tracing.go:   // exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
./backend/tracing.go:   trace_provider := sdktrace.NewTracerProvider(
./backend/tracing.go:       sdktrace.WithSampler(sdktrace.AlwaysSample()),
./backend/tracing.go:       sdktrace.WithBatcher(exporter),
./backend/tracing.go:       sdktrace.WithResource(resource.NewWithAttributes(semconv.SchemaURL, semconv.ServiceName(name))),
./backend/tracing.go:   otel.SetTracerProvider(trace_provider)
./backend/tracing.go:   return trace_provider, nil
./backend/go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0 h1:TKf2uAs2ueguzLaxOCBXNpHxfO/aC7PAdDsSH0IbeRQ=
./backend/go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace v1.14.0/go.mod h1:HrbCVv40OOLTABmOn1ZWty6CHXkU8DK/Urc43tHug70=
./backend/go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0 h1:3jAYbRHQAqzLjd9I4tzxwJ8Pk/N6AqBcF6m1ZHrxG94=
./backend/go.sum:go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp v1.14.0/go.mod h1:+N7zNjIJv4K+DeX67XXET0P+eIciESgaFDBqh+ZJFS4=
./backend/go.sum:go.opentelemetry.io/otel/trace v1.14.0 h1:wp2Mmvj41tDsyAJXiWDWpfNsOiIyd38fy85pyKcFq/M=
./backend/go.sum:go.opentelemetry.io/otel/trace v1.14.0/go.mod h1:8avnQLK+CG77yNLUae4ea2JDQ6iT+gozhnZjy/rw9G8=
./backend/app.go:   sdktrace "go.opentelemetry.io/otel/sdk/trace"
./backend/app.go:func newApp(ctx context.Context, tp *sdktrace.TracerProvider) (*App, error) {
./backend/main.go:  "go.opentelemetry.io/otel/trace"
./backend/main.go:      span := trace.SpanFromContext(ctx)
./backend/main.go:          log.Printf("Error shutting down tracer provider: %v", err)
./files/config.yml:    traces:
./kubernetes/otelcollector-config.yml:        traces:
```

`--exclude` and `--exclude-dir` can be compined as well. Let's remove
`./backend/tracing.go` and `go.mod` and `go.sum` from our results using
`--exclude`.

```bash
grep -r --exclude-dir 'node_modules' --exclude 'tracing.go' --exclude '*.sum' --exclude '*.mod' trace
```

```
./backend/db.go:  sdktrace "go.opentelemetry.io/otel/sdk/trace"
./backend/db.go:func dbConn(ctx context.Context, tp *sdktrace.TracerProvider) (*mongo.Client, error) {
./backend/app.go: sdktrace "go.opentelemetry.io/otel/sdk/trace"
./backend/app.go:func newApp(ctx context.Context, tp *sdktrace.TracerProvider) (*App, error) {
./backend/main.go:    "go.opentelemetry.io/otel/trace"
./backend/main.go:        span := trace.SpanFromContext(ctx)
./backend/main.go:            log.Printf("Error shutting down tracer provider: %v", err)
./files/config.yml:    traces:
./kubernetes/otelcollector-config.yml:        traces:
```

Hopefully this gives you a feel for how you can combine some of the different
flags and features of grep to pinpoint what you're looking for. But that's just
half of the story for grep. In addition to searching files, it is just as
capable at searching anything that you throw at it coming from STDIN.

## Examples - Searching STDIN

### Filtering command output

When you use the pipline operator `|`, the output of the previous command is fed
as input to the next command. For grep, this means that you can search the
results of anything that produces output.

One example I've used recently is when trying to figure out which docker
containers I have downloaded on my system. `docker image ls` just shows me the
complete list, but if I have hundreds of images downloaded it can be quite a
bear to look through them all to find the one I'm looking for. Grep comes to the
rescue! If I'm looking only for which versions of python I have available, I can
filter the results of `docker image ls` down using grep.[^5]

```bash
docker image ls | grep -h python | cut -w -f 1,2
```

```
python    3.7
python    latest
```

### Filtering streams

Because of the wonderful magic of streams combined with
[pipelines](https://en.wikipedia.org/wiki/Pipeline_(Unix)), you can grep the
output of a command that may run for a long time (or forever). Say you are
trying to monitor a log file that\'s being continuously written to. You can use
[tail](https://man7.org/linux/man-pages/man1/tail.1.html) `-f` to follow the
contents and display each new line as soon as it's written to the file, but
there might be a lot of noise is this particular log file. You can filter the
contents in mid stream by piping tail to grep.

```bash
tail -f tmp.log | grep -h -E 'WARNING|ERROR|FATAL'
```

```
[2023-07-15T19:23:31.968712+00:00] app.ERROR: Something went wrong.
[2023-07-15T19:23:32.205448+00:00] app.ERROR: Something slightly worse went wrong.
[2023-07-15T19:23:34.422216+00:00] app.WARNING: Could not find file, I'm freaking out man.
[2023-07-15T19:23:35.382214+00:00] app.FATAL: Something really bad happened, crashing now.
```

With the power of imagination you can see each line appearing as it is written
to the log. Here we're using the `-E` flag which allows us to use regular
expressions for our pattern, allowing us to only print logs with severity
'WARNING' or 'ERROR' or 'FATAL'.

With this, we can filter results as they're streamed to only `warning` or higher
severity logs. But sometimes we want to see wore context before a fatal log
occurs. Simple filtering out `info` and `warning` logs might have conceiled the
problem. Let's use `--before-context` to see what happened before that fatal
event.

```bash
grep --before-context=4 FATAL tmp.log
```

```
[2023-07-15T19:23:33.542697+00:00] app.DEBUG: I'm just going to delete this file, this is fine.
[2023-07-15T19:23:34.101258+00:00] app.DEBUG: Buisiness as usual.
[2023-07-15T19:23:34.223435+00:00] app.DEBUG: Trying to open file.
[2023-07-15T19:23:34.422216+00:00] app.WARNING: Could not find file, I'm freaking out man.
[2023-07-15T19:23:35.382214+00:00] app.FATAL: Something really bad happened, crashing now.
```

## Conclusion

Because grep can operate on any input sent to it, the possible ways to use it
are endless. If you ever find yourself in a situation where you wish you could
filter some data, you should probably reach for grep first and see if it meets
your needs. There are other searching and filtering tools out there that are
more specialized, but grep works everywhere, and it's installed everywhere.

I would encourage you to read the whole grep
[man](https://man7.org/linux/man-pages/man1/man.1.html) pages,
[here](https://man7.org/linux/man-pages/man1/grep.1.html). I covered most of the
common use cases, but there are many more flags and features hidden away in grep
for those willing to search for them.

Notably, this article did not cover any of the variations of grep like `pgrep`,
`rgrep`, `egrep`, `fgrep`, etc. You can read more about them in the grep manual.

# Footnotes

[^1]: <https://en.wikipedia.org/wiki/Ed_(text_editor)>

    Despite being notoriously difficult to understand, ed went on to influence
    and spawn ex, then vi, leading to vim and neovim.

[^2]: One thing to note is that file expansion done through globbing is done
    prior to feeding any arguments to grep; this doesn\'t matter in this case
    but it\'s something to keep in mind.

[^3]: Another thing to note is that shell filename expansion (globbing) does not
    use regular expressions. `*.go` would be an illegal regular expression
    because `*` is a
    [quantifier](https://learn.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference?redirectedfrom=MSDN#quantifiers)
    and must come after some character class expression (like `.`). In shell
    filename expansion, `*` matches every filename. `*.go` matches every file
    that ends with `.go` (note the `.` is not a special character like it is in
    regex). `?` matches any one character (this is exactly like `.` in regular
    expressions).

[^4]: If you\'re ever unsure of the result of a shell glob, you can run
    [echo](man:echo) or [ls](man:ls) first to see which files will be selected.

[^5]: I\'m using the `-h` flag for `grep` which surpresses the
    \'filename\' prefix of its output. When grep is used in a pipeline instead
    of the filename, it prefixes every line with \'(standard input):\' which is
    not super helpful.

    I\'m also using [cut](man:cut) to just show the columns I care about. This
    is mostly so that the output shows up nicely in this blog; it\'s not
    strictly necessary.
