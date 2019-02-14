---
layout: blog_post
title: "GO modifying command agruments"
description: "Tip how to modify command agurment before further processing."
tags:
  - golang
  - command
---

Today I learned something new about golang I want to share: You can modify
arguments to the application once the application is started.

Usually, when designing the command line arguments, arguments have to be parsed,
matched and processed. In golang, this is easily done via importing the
`os` module and using the `os.Args` slice,
which contains every argument passed to the command.
It is simple, just like in the example

```golang
package main

import "os"

func main() {
    argumentsCount := len(os.Args)
    if argumentsCount < 99 {
        doSomething()
    }
    for _, arg := range os.Args {
        process(arg)
    }
}
```

This is fairly trivial until you decide to design a serious command line utility.
Of course, the golang has a nice `flag` package which offers command line flag
parsing and there is 3rd party pflag package for better flag parsing.
And if you are designing big tool, you can take advantage of the cobra package,
which allows you to build tools with git like usage, quite easily.

But what if you need to hack your way around and modify the command arguments
before it is even used in the application? Well, this is what I had to solve.
I had to modify arguments passed to the application before the cobra took control.
For the record, this is not best practice and you should do it only for testing
and developing, not in production.

As I mentioned in the beginning, the arguments are passed to the application
through the `os.Args`, which is a slice. It occurred to me, that slice can be
modified. So I tried it and whoaa, it worked. I was able to modify the command
arguments, even though I always considered it read-only.
The only trick to do it before I called cobra, was to do it in the init phase
before cobra was initialized. The full snippet is below.

```golang

package main

import os

func main() {
    initCobraAndDoStuff()
}

func init() {
    // do the argument editing here

    if os.Args[1] == "test" {
        os.Args = append(os.Args, "all")
    }

    // now the command arguments are:
    // commandName test all
}
```
