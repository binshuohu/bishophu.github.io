---
comments: True
---
I'm going to shift from messing with the scala REPL to writing some serious code as an exercise to apply stuff I've learned so far. Clearly, learning how to use a serious build system is a reasonable start.

I'm from a C++ background and I don't even know how to utilize the power of GNU make or cmake or any other C++ build system. I expect that it's gonna take a little while for me to understand the mechanism behind [SBT](http://www.scala-sbt.org/). Anyway, I'll just use this blog post as a learning note to explain to myself those concepts I find confusing in the official tutorial.

The first thing I find rather vague in SBT is [scopes](http://www.scala-sbt.org/0.13/tutorial/Scopes.html). The tutorial uses the word axes to explain how it works, which I don't find helpful at all. Per the documentation, a key could be associated with more than one scope. It's just like in real life the same phrase could mean complete different things. "I'm going to work, take care of the baby for me, darling." and "He's a snitch, take care of him, Donnie." mean the opposite, right? So a key can have different meanings in different scopes(contexts), but it has only one meaning in a certain scope, which is identified by three axes, namely `Project`, `Configuration` and `Task`.

A key's name is not its whole identiy, so when you want to specify a key, you need three extra parts, one from each axis, to eliminate ambiguity. Axis is kind like namespace, but not exactly, because the scoping rules in SBT do not necessarily support nesting.

When setting a key value in build definition, you can just use a simple key name, in which case you will actually set a key in the default axes, or you can make your intent clear by specifying the exact scope. For example:

```scala
val villain = sbt.settingKey[String]("poor souls who will die at the end of the book")

villain := "Sauron"
villain in (ThisBuild, Compile) := "Saruman"
```

OK, let's break this down and try to make sense of it. In the first line of the snippet, we defined a key named `villain` in the hope to spoiler the whole trilogy of TLOR for those who haven't watched it yet. Well, this example may not make much sense, but I just need two diffenret conspicuous icon to illurstrate how scoping works in SBT.

Now type `sbt` in your console to enter the interactive REPL. Then type `inspect villain`, the output should be like this:

> inspect villain

> [info] Setting: java.lang.String = Sauron

> [info] Description:

> crooked mind

> [info] Provided by:

> {file:/Users/binshuo/repo/practice/}root/*:villain

> [info] Defined at:

> /Users/binshuo/repo/practice/build.sbt:7

> [info] Delegates:

> *:villain

> {.}/*:villain

> */*:villain

> [info] Related:

> {.}/compile:villain[info] [info] [info] [info] [info] [info] [info] 

As you can see, the value to the key `villain` is `Sauron`. Why? Well, when you just give a bare key name without any axes specification, SBT will assume you use default axes. For `Project` axis, it's `current project`, for `Configuration` it's `Global`, and for `Task` it's `Gloabl` as well. The same applys when giving a key a value, just like we defined `villain` as `Sauron` above.

Moving on, since we defined `villain in (ThisBuild, Compile) := "Saruman"`, when we type `inspect {.}/compile:villain`, we get `Saruman`.

But Saruman isn't all mighty, he has limitation. If we try to inspect `{.}/*:villain`, we get nothing because in this case, we specify the configuration axis as `Global` using an asterisk, which is beyond Saruman's reach.

When SBT couldn't find a key in the specified scope, it will try to relax each axis one by one to locate the key name you asked. Try `inspect test:villain`, then observe the Delegates section and try to figure out how SBT relaxes each axis to search for the key.
