+++
date = '2023-12-08T16:00:00-06:00'
draft = false
title = 'jq'
show_reading_time = true
tags = ['how-to']
+++

# Overview

[jq](https://jqlang.github.io/jq/) is a utility to search through, filter, and
modify `json` content.[^1] You can use it to find a value in a deeply nested
object, filter to only objects which contain a field, map the inner contents of
arbitrary data into a flat list, and much much more.

On mac, it can be installed through homebrew:

```bash
brew install jq
```

As an introductory example, say you have the following json input:[^2]

```bash
echo '{"Instances": [{"InstanceId": "sdb3871osdhv89044", "Name": "server0",
"CreationDate": "2021-06-20T03:50:34"},{"InstanceId": "abc123eassa12874", "Name": "server1",
"CreationDate": "2023-11-01T15:13:06"}, {"InstanceId": "aes54iasg41226c",
"Name": "server2", "CreationDate": "2023-11-02T12:07:54"}, {"InstanceId":
"flng45799saooh987", "Name": "server3", "CreationDate": "2023-11-06T21:45:27"}], "ResponseCode": 200}' | tee ~/test.json
```

```
{"Instances": [{"InstanceId": "sdb3871osdhv89044", "Name": "server0",
"CreationDate": "2021-06-20T03:50:34"},{"InstanceId": "abc123eassa12874", "Name": "server1",
"CreationDate": "2023-11-01T15:13:06"}, {"InstanceId": "aes54iasg41226c",
"Name": "server2", "CreationDate": "2023-11-02T12:07:54"}, {"InstanceId":
"flng45799saooh987", "Name": "server3", "CreationDate": "2023-11-06T21:45:27"}], "ResponseCode": 200}
```

Jq with no additional arguments will parse, format, and highlight incoming json:

```bash
cat ~/test.json | jq
```

```json
{
  "Instances": [
    {
      "InstanceId": "sdb3871osdhv89044",
      "Name": "server0",
      "CreationDate": "2021-06-20T03:50:34"
    },
    {
      "InstanceId": "abc123eassa12874",
      "Name": "server1",
      "CreationDate": "2023-11-01T15:13:06"
    },
    {
      "InstanceId": "aes54iasg41226c",
      "Name": "server2",
      "CreationDate": "2023-11-02T12:07:54"
    },
    {
      "InstanceId": "flng45799saooh987",
      "Name": "server3",
      "CreationDate": "2023-11-06T21:45:27"
    }
  ],
  "ResponseCode": 200
}
```

# Filters

## Basic filters

Basic filters can be applied in order to grab specific fields from your input:

```bash
cat ~/test.json | jq '.Instances'
```

```json
[
  {
    "InstanceId": "sdb3871osdhv89044",
    "Name": "server0",
    "CreationDate": "2021-06-20T03:50:34"
  },
  {
    "InstanceId": "abc123eassa12874",
    "Name": "server1",
    "CreationDate": "2023-11-01T15:13:06"
  },
  {
    "InstanceId": "aes54iasg41226c",
    "Name": "server2",
    "CreationDate": "2023-11-02T12:07:54"
  },
  {
    "InstanceId": "flng45799saooh987",
    "Name": "server3",
    "CreationDate": "2023-11-06T21:45:27"
  }
]
```

You can nest filters to retrieve deeply nested data from your input too:

```bash
cat ~/test.json | jq '.Instances[1].Name'
```

```json
"server1"
```

Array access works as expected with zero-based integer indexing. In addition to
simply getting date from input, you can create arbitrary new objects by wrapping
your filters in an object:

```bash
cat ~/test.json | jq '{"ServerName": .Instances[1].Name, "Created": .Instances[1].CreationDate }'
```

```json
{
  "ServerName": "server1",
  "Created": "2023-11-01T15:13:06"
}
```

You can link multiple filters and transformations through the pipe `|` operator.

```bash
cat ~/test.json | jq '.Instances[1] | { "ServerName": .Name, "Created": .CreationDate }'
```

```json
{
  "ServerName": "server1",
  "Created": "2023-11-01T15:13:06"
}
```

Instead of selecting only an individual record in `Instances`, you can execute a
filter over _all_ instances by omitting the offset to the array access.

```bash
cat ~/test.json | jq '.Instances[] | { "ServerName": .Name, "Created": .CreationDate }'
```

```json
{
  "ServerName": "server0",
  "Created": "2021-06-20T03:50:34"
}
{
  "ServerName": "server1",
  "Created": "2023-11-01T15:13:06"
}
{
  "ServerName": "server2",
  "Created": "2023-11-02T12:07:54"
}
{
  "ServerName": "server3",
  "Created": "2023-11-06T21:45:27"
}
```

The results of our filters are streamed one at a time through each pipelined
expression. You can see this by looking at the output; each object is the result
of the second filter (which creates a new object). Further filters in a
subsequent pipeline would work with the new object we created (with keys:
`ServerName` and `Created`).

We can collect the results of all the expressions in the stream by just wrapping
it in an array, similar to how we created the object above.

```bash
cat ~/test.json | jq '[ .Instances[] | { "ServerName": .Name, "Created": .CreationDate } ]'
```

```json
[
  {
    "ServerName": "server0",
    "Created": "2021-06-20T03:50:34"
  },
  {
    "ServerName": "server1",
    "Created": "2023-11-01T15:13:06"
  },
  {
    "ServerName": "server2",
    "Created": "2023-11-02T12:07:54"
  },
  {
    "ServerName": "server3",
    "Created": "2023-11-06T21:45:27"
  }
]
```

## Selection

Often you want to restrict the results not (only) to certain fields of some json
object, but to fields matching some criteria. The jq filter `select`
accomplishes this task. Say we wanted to filter instances to only those which
were created on or after November third:

```bash
cat ~/test.json | jq '.Instances[] | select(.CreationDate > "2023-11-03")'
```

```json
{
  "InstanceId": "flng45799saooh987",
  "Name": "server3",
  "CreationDate": "2023-11-06T21:45:27"
}
```

This can be useful as well on simpler data as well. If given a list of numbers,
you can filter them using `select`:

```bash
echo '[1, 2, 3, 4]' | jq 'map(select(. > 2))'
```

```json
[
  3,
  4
]
```

_NOTE_: `map` is just another way to execute a filter on every element of a
list, you've already seen this presented anonter way: `[ .[] | select(. > 2) ]`
(e.g. create an array `[` from the generated stream of items `.[]` from which
`|` only elements that are `select(` greater than 2 `. > 2` are retained)

# Modifying objects

If there's a need to transform your data by adding fields, `+` is more than up
to the task. In this case I don't want to filter which instances are deemed
"old", instead, I just want a field to indicate _if_ it is old.

```bash
cat ~/test.json | jq '.Instances[] | . + { IsSuperOld: (.CreationDate < "2022-01-01") }'
```

```json
{
  "InstanceId": "sdb3871osdhv89044",
  "Name": "server0",
  "CreationDate": "2021-06-20T03:50:34",
  "IsSuperOld": true
}
{
  "InstanceId": "abc123eassa12874",
  "Name": "server1",
  "CreationDate": "2023-11-01T15:13:06",
  "IsSuperOld": false
}
{
  "InstanceId": "aes54iasg41226c",
  "Name": "server2",
  "CreationDate": "2023-11-02T12:07:54",
  "IsSuperOld": false
}
{
  "InstanceId": "flng45799saooh987",
  "Name": "server3",
  "CreationDate": "2023-11-06T21:45:27",
  "IsSuperOld": false
}
```

The above creates a stream of each object in the field `Instances`, and for
each, adds to it the field `IsSuperOld`, setting it to true or false depending
on if the `CreationDate` of that instance is older than January 1st, 2022.

Assignment in jq is very powerful; in this case, we added the new field
individually onto each element in the stream, but `+` (along with the other
assignment operators) can assign values onto a path wich points to _multiple_
locations. As an example, we can actually simplify the above query like so:

```bash
cat ~/test.json | jq '.Instances[] + { IsSuperOld: (.CreationDate < "2022-01-01") }'
```

```json
{
  "InstanceId": "sdb3871osdhv89044",
  "Name": "server0",
  "CreationDate": "2021-06-20T03:50:34",
  "IsSuperOld": true
}
{
  "InstanceId": "abc123eassa12874",
  "Name": "server1",
  "CreationDate": "2023-11-01T15:13:06",
  "IsSuperOld": true
}
{
  "InstanceId": "aes54iasg41226c",
  "Name": "server2",
  "CreationDate": "2023-11-02T12:07:54",
  "IsSuperOld": true
}
{
  "InstanceId": "flng45799saooh987",
  "Name": "server3",
  "CreationDate": "2023-11-06T21:45:27",
  "IsSuperOld": true
}
```

This is even more powerful when combined with a reducing filter like select:[^3]

```bash
cat ~/test.json | jq '.Instances[] | select(.InstanceId | startswith("a") ) |= . + { IsSuperOld: (.CreationDate < "2022-01-01") }'
```

```json
{
  "InstanceId": "sdb3871osdhv89044",
  "Name": "server0",
  "CreationDate": "2021-06-20T03:50:34"
}
{
  "InstanceId": "abc123eassa12874",
  "Name": "server1",
  "CreationDate": "2023-11-01T15:13:06",
  "IsSuperOld": false
}
{
  "InstanceId": "aes54iasg41226c",
  "Name": "server2",
  "CreationDate": "2023-11-02T12:07:54",
  "IsSuperOld": false
}
{
  "InstanceId": "flng45799saooh987",
  "Name": "server3",
  "CreationDate": "2023-11-06T21:45:27"
}
```

Granted these examples are becoming more and more contrived, but let's break
down what's happening. We want a stream of `Instances`, and for elements
selected by the filter `.InstanceId | startswith("a")`, add the field
`IsSuperOld` which is set to true if the `CreationDate` is before `2022-01-01`.

# Custom functions

If you find yourself reusing a particular filter often, it can be helpful reduce
repitition by storing that filter in one central place. Jq allows this with
named filters, which is calls
[functions](https://jqlang.github.io/jq/manual/#defining-functions). In the
examples above, we could name our \"super old\" filter to avoid repeating
ourselves.

```bash
cat ~/test.json | jq 'def isold: .CreationDate < "2022-01-01"; .Instances[] + { IsSuperOld: isold }'
```

```json
{
  "InstanceId": "sdb3871osdhv89044",
  "Name": "server0",
  "CreationDate": "2021-06-20T03:50:34",
  "IsSuperOld": true
}
{
  "InstanceId": "abc123eassa12874",
  "Name": "server1",
  "CreationDate": "2023-11-01T15:13:06",
  "IsSuperOld": true
}
{
  "InstanceId": "aes54iasg41226c",
  "Name": "server2",
  "CreationDate": "2023-11-02T12:07:54",
  "IsSuperOld": true
}
{
  "InstanceId": "flng45799saooh987",
  "Name": "server3",
  "CreationDate": "2023-11-06T21:45:27",
  "IsSuperOld": true
}
```

# Conclusion

Like usual, we've only scratched the surface of what jq can do. If their
extensive [docs](https://jqlang.github.io/jq/manual/) with included examples
can't help you, chances are there's a stack overflow post from somebody asking
how to do what you're trying to do.

I've personally found it very useful when hacking scripts together which utilize
the aws cli. Because the result its queries are json, the two make a great team.[^4]

Jq does one small job, filter and transform json. Toward this end, it's been
extremely helpful for putting together small bash scripts without having to
reach for something heavier like python and the `json` module. It falls into the
category of: if you need it, you'll know.

# Footnotes

[^1]: Many of these examples are taken from jq's
    [tutorial](https://jqlang.github.io/jq/tutorial/). It's fairly short, I
    recommend you read it.

[^2]: [tee](https://www.man7.org/linux/man-pages/man1/tee.1.html) is used to
    print to stdout as well as send to the provided file.

[^3]: This example uses the update assignment operator
    [|=](https://jqlang.github.io/jq/manual/#update-assignment) which runs its
    input through the filter on the right, and returns the updated result. This
    differs from plain assignment,
    [=](https://jqlang.github.io/jq/manual/#plain-assignment), which overwrites
    its input with the result of the expression on the right. This is important
    in this case since I still wanted all the instances, not just those selected
    by the `startswith` filter.

[^4]: The AWS cli supports the `--query` flag, which accepts a json query syntax
    similar to, but not quite exactly compatible with jq called
    [jmespath](https://jmespath.org). This could be a great option for you, but
    I am already familiar with jq and didn't want to learn _another_ json query
    syntax, so I just pipe the results of my aws calls to jq.
