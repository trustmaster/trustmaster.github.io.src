---
tags: [
    "fbp",
    "architecture",
]
categories: [ "Flow-based Programming" ]
image: "/img/post-bg.jpg"
title: "Extending Flow-based Programming"
description: ""
draft: true
comments: true
date: "2032-07-17T17:30:12+02:00"
---

## Extending Flow-based Programming

This section covers a few practical areas where original Flow-based Programming notation and libraries fall short. But since these issues appear in real world tasks, it can be reasonable to extend Flow-based Programming to cover them.

### Extending component behavior

One very specific problem that exists in FBP is lack of a standard way of extending behavior of existing components. Something that in Object-Oriented Programming is typically achieved with inheritance.

A graph-based solution is based on aggregation: a reusable component is put inside of another graph that contains adapter and extender components. A smallest possible graph looks something like:

```dot
INPORT=Adapter.IN:IN
OUTPORT=Extender.OUT:OUT

Adapter -> OriginalComponent -> Extender
```

Real world "extender graphs" get quite bulky and contain very tricky machinery of adapters.

Typical workarounds for this problem include using the features that a programming language offers: e.g. moving shared code to a reusable "library" or "core package", or actually using inheritance.

### Generic subgraphs

Native language features can be used to build reusable components, but what about subgraphs? A typical case is when you need the same graph but with a slightly different component inside of it. Consider this example piece of an FBP graph:

*ParseJsonFile.fbp*

```dot
GetUrl(cli/Input) -> NAME Reader(fs/ReadFile) -> TEXT Parser(json/Parser)
```

Now what if you need another graph that uses `http/ReadFile` as a reader and everything else stays the same? In existing FBP frameworks you need to duplicate the entire graph.

A potential solution to this could be a way to extend FBP graphs and replace some components. An example graph could look something like:

*ParseJsonUrl.fbp*

```dot
EXTENDS ParseJsonFile
Reader(http/ReadFile)
```

In order for this to work, `Reader` components should implement the same interface.

### Message (IP) types

In most FBP frameworks written in statically typed languages have type definitions for ports in components and have type checks on the component level. It works during static analysis (compile time) and while the data is being sent (run time).

Frameworks written in dynamically typed languages (e.g. ECMAScript) may only have limited type checks. E.g. TypeScript adds some static compile-time checks, but at run time it is very easy to send whatever to a nicely "typed" component.

When it comes to the graph level (be it .fbp, .json, or .drw, or any kind of other textual or visual notation), there are no type definitions there. In smaller applications this is totally fine, because an engineer can check particular component code to find the types of messages that it receives or sends. In larger applications built by multiple people or multiple teams the message types become more crucial. If team B looks at a graph where they need to connect to a component built by team A, how do they know what type of a message the A's component expects without diving deep into its source files?

Adding type information to the graph level can be useful in such complex applications. In some cases it would be a matter of the IDE fetching that information from the source files. In some cases an IDE and local files may be insufficient.

For example, if the application is distributed, and components actually talk to each other via a message queue. In the perfect world, the runtime/orchestrator system should know what type a certain connection has. And it should check if the messages being sent actually comply with that type.

An implementation can use existing schema protocols, e.g. [Protobuf](https://developers.google.com/protocol-buffers).
