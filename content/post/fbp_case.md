+++
tags = [
    "fbp",
    "architecture",
]
categories = [ "Flow-based Programming" ]
image = "/img/post-bg.jpg"
title = "The real case for Flow-based Programming"
description = "The place for Flow-based Programming in Software Engineering and Architecture"
draft = true
comments = true
date = "2018-08-05T17:30:12+02:00"
+++

Coding is about making software that works. Optimization is about making software work well. Design is about making software maintainable over time by teams of human beings.

A common learning path looks like this: first you learn to code, then you learn to write code that behaves well, then you learn design patterns and start applying them reflexively, then you learn to make reasonable design decisions. And then you don't write code anymore, because you write documents and diagrams. Or because you have retired.

Jokes apart, design and coding usually come separately. So much separately that in a couple years of a project lifetime they have nothing in common. It's a serious problem that is either ignored or tackled by great minds like [Simon Brown in this talk](https://www.youtube.com/watch?v=GAFZcYlO5S0).

Then, what if I told you that somewhere in a parallel universe this problem doesn't exist? In that universe they design the code and code the design. If you don't believe that universe exists, then it's a good sign of its main problem: it's not wide-spread in the software industry.

Let's reveal that universe to more of our kind: Flow-based Programming.

## What is Flow-based programming?

[Flow-based Programming (FBP)](https://en.wikipedia.org/wiki/Flow-based_programming) is a combination of [Dataflow programming](http://en.wikipedia.org/wiki/Dataflow_programming) and [Component-based software engineering](http://en.wikipedia.org/wiki/Component-based_software_engineering) built around the idea of Components communicating via messages (Information Packets, IPs) sent through Connections. So, it makes a program look somewhat like this:

![NoFlo graph example](/img/post/noflo_app_example.png)

What makes Flow-based Programming different from other approaches is:

 - Dataflow: your application is how the data flows and what operations change it. Rather than what command to execute in which case.
 - 2 engineering perspectives: Graph and Component. Components are just pieces of traditional code. Graphs define how data travels between Components. Any Graph is a Component itself and may include other Graphs.
 - Components have input and output Ports. This is how they are seen and used from outside.
 - Components run concurrently and share no state. But inside each component can be either stateful or stateless.
 - The messages (IPs) are more self-contained objects than just primitive values, which makes communication and synchronization more effective.

If you are into studying FBP, I recommend you to start with the [Flowbased wiki](https://github.com/flowbased/flowbased.org/wiki).

## Where is it applicable?

Flow-based programming applies most naturally to domains which already have the notion of components, black boxes, processes, pipelines, etc. In particular, FBP is very attractive in:

 - Bioinformatics
 - Data, e.g. ETL
 - DevOps
 - IoT
 - Machine Learning

But being a general-purpose programming paradigm, it is also being applied to other domains such as:

 - Business applications
 - Web services

Although in the latter it is way harder to shift from traditional way of thinking in classes, MVC, etc. and to build the application truly around the data flow, without sacrificing its maintainability. Worth noticing nevertheless, that [originally FBP was applied](http://www.jpaulmorrison.com/fbp/history.html) to transactional Banking software where it was used rather successfully for a good sequence of years.

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

With FPB, you write low level components in conventional textual programming languages, making that tool do its best job and encapsulating implementation details.

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

And there goes distributed transactions problem as well. FBP hasn't escaped it automagically.

### Having no code to write

The other of the false FBP expectations is that the community can write such a library of reusable components, that later on there is very few code to be written: you'd drop the existing components into your graphs and that's it.

This assumption made people concentrate on writing components which are either too small or too generic. Which made neither of them easy to use. With very fine-grained components your graphs get too big and too complicated. With very generic components you end up writing a lot of adapters and workarounds. So, when it comes to component scale, the golden mean works the best.

But that, at least nowadays, forces you to write many components specific to your application. However, in the Object Oriented Programming universe the gurus have come to a similar conclusion: duplicate code is not such a bad thing in the end.

## Can I use FBP today?

Yes, e.g.

 - with Java: [JavaFBP](https://github.com/jpaulm/javafbp) + [DrawFBP](https://github.com/jpaulm/drawfbp)
 - with JavaScript: [NoFlo](https://noflojs.org/) + [Flowhub](https://flowhub.io/)
 - with Go: [GoFlow](https://github.com/trustmaster/goflow), [Flowbase](https://github.com/flowbase/flowbase), [Cascades](https://github.com/cascades-fbp/cascades)
 - with Rust: [Fractalide](https://github.com/fractalide/fractalide)
 - with Python: [multiple packages](https://wiki.python.org/moin/FlowBasedProgramming)
 - probably with your other favorite language, Google knows how many Flow-based Programming runtimes are around.

 Will it be easy? Maybe yes, but probably not. Unfortunately we are not in a state when we can just throw all the UML and piles of classes away and switch to Graphs and Components all of a sudden. There are some important problems to be solved first. In particular, the learning curve may be steep both because of the paradigm shift and because of maturity of existing tools.

I'll follow up with an article addressing the next milestones that we need to reach to make FBP usable for everyone.