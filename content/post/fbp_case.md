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
title = "The real case for Flow-based Programming"
description = "The place for Flow-based Programming in Software Engineering and Architecture, and how to get it right there"
draft = true
comments = true
date = "2018-08-05T17:30:12+02:00"
+++

Recently I have watched a lot of software engineers and software architects struggle in the wild. Trying to design good software, write good code, and make it evolve over time without turning into a Big Bowl of Spaghetti. And despite having ultimate methodologies like [DDD, Clean Architecture and others](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/), we still struggle a lot with software design and its consequences. [This talk](https://www.youtube.com/watch?v=GAFZcYlO5S0) by Simon Brown provides a good example of that.

Somewhere in a parallel universe of [Flow-based programming](https://en.wikipedia.org/wiki/Flow-based_programming) there are people who know how to make software design over time together with code, but struggle making their ideas applicable and wide-spread among other engineers and companies.

Being aware of both, here is what I have to say.

But first,

## What is Flow-based programming?

[Flow-based Programming (FBP)](https://en.wikipedia.org/wiki/Flow-based_programming) is a combination of [Dataflow programming](http://en.wikipedia.org/wiki/Dataflow_programming) and [Component-based software engineering](http://en.wikipedia.org/wiki/Component-based_software_engineering) built around the idea of Components communicating via messages (Information Packets, IPs) sent through Connections. So, it makes a program look somewhat like this:

![NoFlo graph example](/img/post/noflo_app_example.png)

What makes Flow-based Programming different from other approaches is:

 - Dataflow: it concentrates on how data flows through the system and what changes it experiences, rather than what command to execute in which case.
 - 2 engineering perspectives/views: Graph (usually visual) and Component (usually textual). Components are written in conventional programming languages, and are wired together in a graph. Graphs can be nested, so you can "zoom" into a specific component and see it as a sub-graph.
 - Components define their interface via incoming and outgoing Ports. This ports interface is all the outside world knows about the component technically.
 - Components run concurrently and share no state. But inside each component can be stateful or stateless.
 - The messages (IPs) are more self-contained objects than just primitive values, which makes communication and synchronization more effective.

If you are into studying FBP, I recommend you to start with the [Flowbased wiki](https://github.com/flowbased/flowbased.org/wiki).

## Where is it applicable?

Flow-based programming applies most naturally to domains which already have the notion of components, black boxes, processes, pipelines, etc. In particular, FBP is very attractive in:

 - Bioinformatics
 - Data, e.g. ETL
 - IoT
 - Machine Learning

But being a general-purpose programming paradigm, it is also being applied to other domains such as:

 - Business applications
 - Web services

Although in the latter FBP practitioners either give up to implementing conventional software patterns on top of FBP, or go against the common thread of how clean software is built and reinvent all the fundamental principles on their own. Worth noticing nevertheless, that [originally FBP was applied](http://www.jpaulmorrison.com/fbp/history.html) to transactional Banking software where it was used rather successfully for a good sequence of years.

## Problems it solves

Call it a programming paradigm, an architecture pattern, whatever. The real problems in Software Engineering that Flow-based Programming solves are, ranked by importance.

### Comprehensibility of Software systems

Humans understand diagrams quicker than hundreds lines of code. Most software projects start on a whiteboard. With FBP, they continue living on a whiteboard, so that at any point of time you can see on a diagram how an application is structured and how it behaves.

### Evolution of Architecture vs. Code

In conventional Software Design, diagrams get abandoned soon after the implementation starts. It's very hard to keep both diagrams and code up to date.

In the opposite approach the diagrams are generated from code, so you can get actual diagrams at any time, but the resulting diagrams are complicated, because they are a side effect of the design process rather than its aim.

With FBP, diagrams _are_ code. Problem solved.

### Granularity of Dataflow systems

The problem of dataflow approach is that it often gets too granular and low level. There are too many details on the diagrams.

With FPB, you write low level components in conventional textual programming languages, making that tool do its best job and hiding unnecessary detail under the hood.

### Isolation of Dataflow languages

Some dataflow languages force you to implement everything either with the visual language itself, or with a low-level language that it is implemented in (e.g. C).

With FBP you don't need to reinvent all the wheels, you just use existing libraries of your favorite language inside the components. Cross-language and remote communication is encouraged by use of [FBP Protocol](https://flowbased.github.io/fbp-protocol/).

## Problems it does not solve

Just like any other thing, Flow-based programming is not a silver bullet. There are some problems people believe it to solve but in fact it doesn't or it does only partially.

### Designing clean software

It doesn't come for granted. Best described with a quote from [Henri Bergius](https://bergie.iki.fi):

> In FBP, your spaghetti code actually looks like spaghetti.

You still have to work hard and apply your best knowledge to design clean software with FBP.

The bad news is that many existing software patterns simply don't work when you change perspective from control flow to data flow. For example, if you try to implement MVC with complex Data Abstraction Layer glued by Dependency Injection, it is going to end up in a mess that you should rather have implemented on top of one of the conventional frameworks. There are few patterns which work effectively, and there are more to be discovered by trial and error.

The good news is that it's way easier to spot that something is wrong with the architecture, because you can actually _see_ it.

### Issues of parallel and distributed systems

Originally it was thought that as all Processes run in parallel, and there is single ownership rule (each Information Packet is only owned by one Process), all concurrency issues are solved inherently.

Some of them are: in FBP you design your program as a concurrent system from the very beginning, so you have to think how the data flows between parallel branches in the graph.

But the coin has the other side: as all Processes may run in parallel, you need a lot of synchronization. Especially in long-running applications processing concurrent requests.

And there goes distributed transactions problem as well. FBP hasn't escaped this problem automagically.

### Having no code to write

The other of the false FBP expectations is that the community can write such a library of reusable components, that later on there is very few code to be written: you'd drop the existing components into your graphs and that's it.

This assumption made people concentrate on writing components which are either too small or too generic. Which made neither of them easy to use. With very fine-grained components your graphs get too big and too complicated. With very generic components you end up writing a lot of adapters and workarounds. So, when it comes to component scale, the golden mean works the best.

But that, at least nowadays, forces you to write many components specific to your application. However, in the Object Oriented Programming universe the gurus have come to a similar conclusion: duplicate code is not such a bad thing in the end.

## What is yet to be done

Unfortunately we are not in a state when we can just throw all the UML and piles of classes away and switch to Graphs and Components all of a sudden. There are some important problems to be solved first.

### Reducing the barrier of entry

At the moment it's hard to start delivering working Flow-based software. It is primarily caused by two factors:

1. There are not so many Flow-based runtimes around that can be used in the existing applications.[Usually FBP runtimes force you to start from scratch.
2. There is no convenient Graph editor. The existing ones are either too old-fashioned, or too complex all-in solutions.

There are of course traditional factors like lack of standards, documentation, and examples. But these I believe to be secondary to the first two problems.

#### Embeddable run-times

FBP runtimes/libraries/frameworks should be easy to embed in existing projects. [NoFlo's asComponent](https://bergie.iki.fi/blog/ascomponent/) is a good example of such embedding feature.

Clean API, documentation, and examples are of undoubted importance. But programmers want to be able to start with what they already know and have.

#### Simple and convenient visual Graph editor

Presently there are two biggest players in this market: [DrawFBP](https://github.com/jpaulm/drawfbp) and [Flowhub](https://flowhub.io/). And they are the very opposites of each other.

DrawFBP is a very old Java app that uses its very own UI flows to create FBP diagrams. It also comes with a code generator feature which works best for the classic JavaFBP. But it doesn't support universal `.fbp` and `.json` graph notations and isn't very useful for anything than just sketching (hello, UML) if you use a runtime that is not explicitly supported by this tool.

Flowhub is an app that runs right in your browser, and it's an IDE which does everything: edits graphs and components, saves projects on GitHub, runs tests, talks to the runtime, introspects the running application. And it does so by making use of universal protocols and file formats. But despite all of these being amazing features and despite my affiliation with Flowhub, it's not what the majority of the Flow-based programming enthusiasts need.

What we need is neither Blender nor Eclipse. What we need is more like SublimeText or Visual Studio Code these days. We want something that can edit `.fbp` and `.json` files in a visually appealing and intuitive way. And that's it for now, the rest can be done by firing up a command in CLI. We want to try things fast locally and see how they work/break. So if we had something that looks like Flowhub but edits local graph files, it would be just enough.

### Discovering best design practices

As I mentioned before, Object Oriented design patterns don't work in Flow-based world. Some of them do, but usually they become ugly. On the other hand, Robert "Uncle Bob" Martin even warned us in his "Clean Architecture" book:

> Never design applications after data flow

Amen. We are all cursed now. What should we do? The first obvious answer is go back to OOP. Period.

The second obvious answer is find patterns that work. Which means writing tons of applications, failing a lot, and coming to conclusions. But before we close this article and start writing a doomed projects, let's remember what problem the layered architectures and software patterns tend to solve: making applications easy to change over time. And in this respect there are two important questions to be answered:

1. How to structure Flow-based projects so that they are easy to navigate and change. The traditional problem of layers, packages, and naming convention.
2. How to make Components extensible. Unfortunately, FBP has no analogue of neither inheritance (so, none of the OOP tricks with interfaces work) nor plug-ins (you can theoretically connect other components to extend behavior, but in practice it is very complicated and barely reusable).

Both questions are worth a full-time research, so I leave them open for now.

## The purpose of this document

Sorry, I didn't mean to say that FBP is bad or immature. I think it is brilliant and the problems it solves overrule the difficulties it may bring.

If you are new to FBP, please go and re-read the first 3 sections of this article. And then proceed to external resources on Flow-based Programming.

If you are one of my fellow FBP practitioners, please read the previous section as an action point or at least food for thought. I've just done it myself.

Thank you!
