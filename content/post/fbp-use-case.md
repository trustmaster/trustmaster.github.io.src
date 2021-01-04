---
tags: [
    "fbp",
    "architecture",
]
categories: [ "Flow-based Programming" ]
image: "/img/post-bg.jpg"
title: "The use cases for Flow-based Programming"
description: "The place for Flow-based Programming in Software Engineering and Architecture"
draft: true
comments: true
date: "2018-08-05T17:30:12+02:00"
---

Coding is about making software that works. Optimization is about making software work well. Design is about making software maintainable over time by teams of human beings.

A common learning path looks like this: first you learn to code, then you learn to write code that behaves well, then you learn design patterns and start applying them reflexively, then you learn to make reasonable design decisions. And then you don't write code anymore, because you write documents and diagrams. Or because you have retired.

Jokes apart, design and coding usually come separately. So much separately that in a couple years of a project lifetime they have nothing in common. It's a serious problem that is either ignored or tackled by great minds like [Simon Brown in this talk](https://www.youtube.com/watch?v=GAFZcYlO5S0) introducing the [C4 diagramming model](https://c4model.com/).

Then, what if I told you that somewhere in a parallel universe this problem doesn't exist? In that universe they design the code and code the design. If you don't believe that universe exists, then it's a good sign of its main problem: it's not wide-spread in the software industry.

Let's reveal that universe to more of our kind: Flow-based Programming.

## What is Flow-based programming?

[Flow-based Programming (FBP)](https://en.wikipedia.org/wiki/Flow-based_programming) is a programming paradigm, which combines [Dataflow programming](http://en.wikipedia.org/wiki/Dataflow_programming) with [Component-based software engineering](http://en.wikipedia.org/wiki/Component-based_software_engineering). In FBP, programs are networks of Components communicating via messages called Information Packets (IPs) sent through Connections. A program can look somewhat like this:

![NoFlo graph example](/img/post/noflo_app_example.png)

This table may help you understand how Flow-based Programming is similar or different from other approaches:

|Criteria      |Flow-based programming|Dataflow programming|Imperative programming|
|--------------|----------------------|--------------------|----------------------|
|Mindset|Data flows through the program and operations transform the data|Data flows through the program and operations transform the data|Operations are executed on data in certain order under certain conditions|
|Program model|A program is a graph of connected components|A program is a graph of connected components|A program is a sequence of instructions|
|Model visualization|Data flow graph|Data flow graph|Control flow chart|
|Representation|Visual (graphs) + Text (components) |Visual (typically), Text (sometimes)| Text|
|Parallelism|Implicit: all operations are parallel by default|Implicit: all operations are parallel by default|Explicit: parallelism needs to be programed|
|Control model|Active: components control their lifecycle|Reactive: components are activated by external events|Active: components control their lifecycle|

What makes Flow-based Programming different from other approaches is:

 - 2 engineering perspectives: Graph and Component. Components describe the behavior of data transformations. They are typically pieces of conventional code written as text files. Graphs define how data travels between Components. Graphs are typically visual or at least written in a declarative language.
 - Any Graph is a Component itself and may include other Graphs. Nesting allows for decomposing the program into comprehensive parts.
 - Component interface consists of input and output Ports. At the same time, Ports are the primary means of interacting with the outside world from inside the Component.
 - Components actively read the incoming data and decide how to react to it. This is different from reactive programming, where reactions are automatically fired on events by the runtime.
 - Components run concurrently and share no state. But inside each component can be either stateful or stateless.
 - The messages (IPs) are typically self-contained objects rather than just primitive values, which makes communication and synchronization more effective.
 - Flow-based systems perform parallel computations by default. All components in the graph are independent and run in parallel.

If you are into studying FBP, I recommend you to start with the [Flowbased wiki](https://github.com/flowbased/flowbased.org/wiki).

## Where is it applicable?

Flow-based programming applies most naturally to domains which already have the notion of components, black boxes, processes, pipelines, etc. In particular, FBP is very attractive in:

- Bioinformatics
- Data engineering, e.g. ETL
- DevOps
- IoT
- Low-code systems
- Machine Learning

> **Is this just guesswork or based on real data?** Sources of these conclusions are:
>- a questionnaire sent on Flow-based programming mailing list a few years ago
>- a questionnaire I posted in GoFlow repository on GitHub
>- my observations around the FBP community on the mailing list and Twitter during the last 10 years
>
> There is no strong statistical model behind it. But given enough motivation, such a research can be conducted. 

Originally introduced as a general-purpose programming paradigm, Flow-based programmng has also been applied to other domains such as:

 - Business applications
 - Web services

However, in the latter case it is way harder to shift from traditional way of thinking to truly benefit from the shift to the Flow-based paradigm. More on that in the next sections. It is worth noticing nevertheless, that [originally FBP was applied](http://www.jpaulmorrison.com/fbp/history.html) to transactional Banking software where it was used rather successfully for a good sequence of years.

## Problems it solves

> **IMO Warning**. Everything that you are going to read below is my personal opinion. I've been participating in the Flow-based Programming community for 10 years, built an academic framework, and a few commercial Flow-based systems that worked in production. All the statements in this and the following sections are based on my own ideas, observations, and fallacies. Trust it for your own risk.

When I started to learn Flow-based programming in 2010, it had a massive wow-effect on me and seemed to me like the ultimate paradigm I want to be using from then till the point when AI replaces programmers at work. That was naive, in multiple ways.

A decade later, I still think Flow-based Programming has several important advantages that the Software industry is missing. But these advantages are not the ones which I anticipated in the beginning. In this section I will share the modern Software Engineering problems (not Programming problems!) that Flow-based Programming successfully solves.

### Comprehensibility of Software systems

Humans understand diagrams quicker than hundreds lines of code. Most software projects start on a whiteboard. With FBP, they continue living on a whiteboard, so that at any point of time you can see on a diagram how an application is structured and how it behaves.

### Evolution of Architecture vs. Code

In conventional Software Design, diagrams get abandoned soon after the implementation starts. It's very hard to keep both diagrams and code up to date.

In the opposite approach the diagrams are generated from code, so you can get actual diagrams at any time, but the resulting diagrams are complicated, because they are a byproduct of the implementation process, rather than a design guidance.

**In FBP, diagrams _are_ code**. Problem solved.

### Granularity of Dataflow systems

The main issue with dataflow approach is that it often gets too granular and low level. It is not uncommon in dataflow systems to have diagrams the size of a tennis-court with hundreds and hundreds of components on them. There are too many details on such diagrams.

With FPB, you write low level components in conventional textual programming languages. Those languages were created to describe behavior, so let the tool do its best job. And then encapsulate implementation details behind the component interface.

You can choose any level of granularity: from atomic operations to large subsystems represented as Components. The optimal choice usually sits somewhere in the middle. 

### Isolation of Dataflow languages

Some dataflow languages force you to implement everything either with the visual language itself, or with a low-level language that it is implemented in (e.g. C).

With FBP you don't need to reinvent all the wheels, you just use existing libraries of your favorite language inside the components. Cross-language and remote communication is encouraged by use of [FBP Protocol](https://flowbased.github.io/fbp-protocol/), meaning that your application can as well be written in different languages or distributed across different machines.

## Problems it does not solve

Just like any other thing, Flow-based programming is not a silver bullet. There are some problems people believe it to solve but in fact it doesn't or it does only partially.

### Designing clean software

It doesn't come for granted. Best described with a quote from [Henri Bergius](https://bergie.iki.fi):

> In FBP, your spaghetti code actually looks like spaghetti.

You still have to work hard and apply your best knowledge to design clean software with FBP.

The bad news is that many existing software patterns simply don't work when you change perspective from control flow to data flow. For example, if you try to implement MVC with complex Data Abstraction Layer glued by Dependency Injection, it is going to end up in a huge mess you won't understand in 5 minutes after you create it. It is better to implement such things on top of the conventional frameworks.

There are few patterns which work effectively (e.g. [Assembly line](https://github.com/noflo/noflo-assembly/wiki/Introduction)), and there are more to be discovered by trial and error.

The good news is that it's way easier to spot when something is wrong with the architecture of a Flow-based system, because you can actually _see_ it.

### Issues of parallel and distributed systems

Originally it was thought that as all Processes run in parallel, and there is single ownership rule (each Information Packet is only owned by one Process), all concurrency issues are solved inherently.

Some of them are: in FBP you design your program as a concurrent system from the very beginning, so you have to think how the data flows between parallel branches in the graph from day one.

But the coin has the other side: as all Processes may run in parallel, you need a lot of synchronization. Especially in long-running applications processing concurrent requests.

And there goes the distributed transactions problem as well. FBP hasn't escaped it automagically.

### Having no code to write

The other of the failed FBP expectations is that the community can write such a library of reusable components, that later on there is very few code to be written. You'd drop the existing components into your graphs and that's it.

This assumption made people concentrate on writing components which are either too small or too generic. Which made neither of them easy to use. With very fine-grained components your graphs get too big and too convoluted. With very generic components you end up writing a lot of adapters and workarounds. So, when it comes to component scale, the golden mean works the best.

But that, at least nowadays, forces you to write many components specifically for your application. However, in the Object Oriented Programming universe the gurus have come to a similar conclusion: duplicate code is not such a bad thing in the end.

## Can I use FBP today?

Yes, e.g.

 - with Java: [JavaFBP](https://github.com/jpaulm/javafbp) + [DrawFBP](https://github.com/jpaulm/drawfbp)
 - with JavaScript: [NoFlo](https://noflojs.org/) + [Flowhub](https://flowhub.io/)
 - with Go: [GoFlow](https://github.com/trustmaster/goflow), [Flowbase](https://github.com/flowbase/flowbase), [Cascades](https://github.com/cascades-fbp/cascades)
 - with Rust: [Fractalide](https://github.com/fractalide/fractalide)
 - with Python: [multiple packages](https://wiki.python.org/moin/FlowBasedProgramming)
 - probably with your other favorite language, Google knows how many Flow-based Programming runtimes are around.

 Will it be easy? Maybe yes, but probably not. Unfortunately we are not in a state when we can just throw all the UML and piles of classes away and switch to Graphs and Components all of a sudden. There are some important problems to be solved first. In particular, the learning curve may be steep both because of the paradigm shift and because of maturity of existing tools.

The remaining part of this article addressing the next milestones that we need to reach to make FBP usable for everyone.

## Next steps for Flow-based Programming adoption

This is my open letter to all FBP practitioners regarding the goals that the community needs to achieve in the nearest future in order for Flow-based Programming to become more widely adopted.

### Reduce the barrier of entry

#### Simple and convenient visual Graph editor

Presently there are two biggest players in this market: [DrawFBP](https://github.com/jpaulm/drawfbp) and [Flowhub](https://flowhub.io/). And they are the very opposites of each other.

DrawFBP is a Java app that was started long ago and its interface isn't very appealing to programmers of the modern days. It uses very own UI flows to create FBP diagrams. It also comes with a code generator feature which works best for the classic JavaFBP. But it doesn't support universal `.fbp` or `.json` graph notations and isn't very useful for anything than just sketching (hello, UML) if you use a runtime that is not explicitly supported by this tool.

Flowhub is an app that runs right in your browser, and it's an IDE which does everything: edits graphs and components, saves projects on GitHub, runs tests, talks to the runtime, introspects the running application. And it does so by making use of [universal protocol](https://github.com/flowbased/fbp-protocol) and file formats. But despite all of these being amazing features and despite my affiliation with Flowhub, it's not what the many of the Flow-based programming enthusiasts need.

What we need is neither Blender nor Eclipse. What we need is more like SublimeText or Visual Studio Code these days. We want something that can edit `.fbp` and `.json` files in a visually appealing and intuitive way. And that's it for now. The rest can be done by firing up a command in CLI. We want to try things fast locally and see how they work/break. So if we had something that looks like Flowhub but edits local graph files, it would be just enough.


#### Run-times effectively integrated with existing software

FBP runtimes/libraries/frameworks should be easy to embed in existing projects. [NoFlo's asCallback](https://noflojs.org/documentation/embedding/) is a good example of such embedding feature.

Clean API, documentation, and examples are of undoubted importance. But programmers want to be able to start with what they already know and have.

### Discovering best design practices

As I mentioned before, Object Oriented design patterns don't work in Flow-based world. Some of them do, but usually they become ugly. On the other hand, Robert "Uncle Bob" Martin even warned us in his "Clean Architecture" book:

> Never design applications after data flow

Amen. We are all cursed now. What should we do? The first obvious answer is go back to OOP. Period.

The second obvious answer is find patterns that work. Which means writing tons of applications, failing a lot, and coming to conclusions. But before we close this article and start writing a doomed projects, let's remember what problem the layered architectures and software patterns tend to solve: making applications easy to change over time. And in this respect there are two important questions to be answered:

1. How to structure Flow-based projects so that they are easy to navigate and change. The traditional problem of layers, packages, and naming convention.
2. How to make Components extensible. Unfortunately, FBP has no analogue of neither inheritance (so, none of the OOP tricks with interfaces work) nor plug-ins (you can theoretically connect other components to extend behavior, but in practice it is very complicated and barely reusable).

The first question speaks for itself and takes completing a few big projects to start recognizing a pattern. The second can be explained further here.

#### Interfaces

Interfaces make true black boxes possible.

![Storage interface](/img/post/fbp_case/order_storage.png)

### The purpose of this document

Sorry, I didn't mean to say that FBP is bad or immature. I think it is brilliant and the problems it solves overrule the difficulties it may bring.

If you are new to FBP, please go and re-read the first 3 sections of this article. And then proceed to external resources on Flow-based Programming.

If you are one of my fellow FBP practitioners, please read the previous section as an action point or at least food for thought. I've just done it myself.

Thank you!

