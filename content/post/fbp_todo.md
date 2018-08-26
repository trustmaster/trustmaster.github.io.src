+++
tags = [
    "fbp",
    "dataflow",
    "design",
    "architecture",
]
categories = [
    "Flow-based Programming",
    "Software Design",
]
image = "/img/post-bg.jpg"
title = "Flow-based Programming: TODO"
description = "What we should do next for FBP to proliferate"
draft = true
comments = true
date = "2018-08-05T17:30:12+02:00"
+++

This is my open letter to all FBP practitioners regarding the goals that the community needs to achieve in the nearest future in order for Flow-based Programming to become more widely adopted.

## Reduce the barrier of entry

### Simple and convenient visual Graph editor

Presently there are two biggest players in this market: [DrawFBP](https://github.com/jpaulm/drawfbp) and [Flowhub](https://flowhub.io/). And they are the very opposites of each other.

DrawFBP is a Java app that was started long ago and its interface isn't very appealing to programmers of the modern days. It uses very own UI flows to create FBP diagrams. It also comes with a code generator feature which works best for the classic JavaFBP. But it doesn't support universal `.fbp` or `.json` graph notations and isn't very useful for anything than just sketching (hello, UML) if you use a runtime that is not explicitly supported by this tool.

Flowhub is an app that runs right in your browser, and it's an IDE which does everything: edits graphs and components, saves projects on GitHub, runs tests, talks to the runtime, introspects the running application. And it does so by making use of [universal protocol](https://github.com/flowbased/fbp-protocol) and file formats. But despite all of these being amazing features and despite my affiliation with Flowhub, it's not what the many of the Flow-based programming enthusiasts need.

What we need is neither Blender nor Eclipse. What we need is more like SublimeText or Visual Studio Code these days. We want something that can edit `.fbp` and `.json` files in a visually appealing and intuitive way. And that's it for now. The rest can be done by firing up a command in CLI. We want to try things fast locally and see how they work/break. So if we had something that looks like Flowhub but edits local graph files, it would be just enough.


### Run-times effectively integrated with existing software

FBP runtimes/libraries/frameworks should be easy to embed in existing projects. [NoFlo's asCallback](https://noflojs.org/documentation/embedding/) is a good example of such embedding feature.

Clean API, documentation, and examples are of undoubted importance. But programmers want to be able to start with what they already know and have.

## Discovering best design practices

As I mentioned before, Object Oriented design patterns don't work in Flow-based world. Some of them do, but usually they become ugly. On the other hand, Robert "Uncle Bob" Martin even warned us in his "Clean Architecture" book:

> Never design applications after data flow

Amen. We are all cursed now. What should we do? The first obvious answer is go back to OOP. Period.

The second obvious answer is find patterns that work. Which means writing tons of applications, failing a lot, and coming to conclusions. But before we close this article and start writing a doomed projects, let's remember what problem the layered architectures and software patterns tend to solve: making applications easy to change over time. And in this respect there are two important questions to be answered:

1. How to structure Flow-based projects so that they are easy to navigate and change. The traditional problem of layers, packages, and naming convention.
2. How to make Components extensible. Unfortunately, FBP has no analogue of neither inheritance (so, none of the OOP tricks with interfaces work) nor plug-ins (you can theoretically connect other components to extend behavior, but in practice it is very complicated and barely reusable).

The first question speaks for itself and takes completing a few big projects to start recognizing a pattern. The second can be explained further here.

### Interfaces

Interfaces make true black boxes possible.

![Storage interface](/img/post/fbp_case/order_storage.png)

## The purpose of this document

Sorry, I didn't mean to say that FBP is bad or immature. I think it is brilliant and the problems it solves overrule the difficulties it may bring.

If you are new to FBP, please go and re-read the first 3 sections of this article. And then proceed to external resources on Flow-based Programming.

If you are one of my fellow FBP practitioners, please read the previous section as an action point or at least food for thought. I've just done it myself.

Thank you!
