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
date: "2022-07-17T17:30:12+02:00"
---

> In memory of [J. Paul Morrison](https://jpaulm.github.io/index.html). May he rest in peace, and his work continue to live.

Coding is about making software that works. Optimization is about making software work well. Design is about making software maintainable over time by teams of human beings.

A common learning path looks like this: first you learn to code, then you learn to write code that behaves well. Then you learn design patterns and start applying them reflexively. Then you learn to make reasonable design decisions. And then you don't write code anymore, because you write documents and diagrams. Or because you have retired. Or you became a manager which is basically the same thing.

Jokes apart, design and coding usually come separately. So much separately that in a couple years of a project lifetime they have nothing in common. It's a serious problem that many people ignore, and many others try to address. But nobody seems to have managed to keep the code and design diagrams in 100% in sync all the time so far.

Then, what if I told you that somewhere in a parallel universe this problem doesn't exist? In that universe they design the code and they code the design. If you don't believe that universe exists, it reveals its main problem: it's not so wide-spread and well adopted.

Let's reveal that universe to more of our kind: Flow-based Programming.

## What is Flow-based Programming?

[Flow-based Programming (FBP)](https://en.wikipedia.org/wiki/Flow-based_programming) is a programming paradigm, which combines [Dataflow programming](http://en.wikipedia.org/wiki/Dataflow_programming) with [Component-based software engineering](http://en.wikipedia.org/wiki/Component-based_software_engineering). In FBP, programs are networks of Components communicating via messages called Information Packets (IPs) sent through Connections.

### Graphs and subgraphs

An FBP program is a graph and it can look somewhat like this:

![NoFlo graph example](/img/post/fbp/noflo_app_example.png)

Each node of this graph is a Component instance called a Process. Think of Components as of Classes in OOP, and of Processes as of Objects. As the name suggests, each Process is executed concurrently and independently of other Processes. Depending on a runtime, concurrency can be implemented with threads, system processes, coroutines, etc.

Data flows through connections from one process to the other. A connection may have a buffer and act as a queue, giving the receiver some time to consume all the messages. Or it can be a blocking connection, meaning that a sender has to wait for receiver to consume the message. For example, in JavaFBP the concept of back pressure is very important.

Each node on the graph can actually be a graph itself, also known as subgraph. This nesting concept of Flow-based Programming is very powerful, because it allows for encapsulating parts of the program in subgraphs and keeping complexity at a moderate level. You can zoom into a subgraph to see more details, and zoom out to see a more high level picture.

### Components

Let's look at what a Component is like:

![ScanKeyword component example](/img/post/fbp/fbp_scankeyword_component.png)

In this example, we have a process called `ScanInport` which is an instance of a component called `ScanKeyword` from the `dsl` package. It has 3 input ports and 1 output port:

- `set` is a configuration input. For this component, it sets a keyword to scan for
- `type` is another configuration input, which assigns a specific type string with that keyword
- `in` is the main input which accepts a stream of incoming packets (messages)
- `out` is the main output port, which sends a stream of outgoing packets (messages)

As you may have guessed by looking at its interface, this process scans the incoming stream for the `INPORT` keyword and if it finds a match it sends the matched token to the output. The strings `INPORT` and `inport` that you see on the graph don't have to come from any other processes. They are called Initial Information Packets (IIP) and are sent once during program's initialization.

What's inside the component? Anything! The only constraint is that it has to read from its input ports and write to the output. Everything else is up to you: what to do with the data, whether to make it stateful or stateless, which libraries to use, persistence, UI, algorithms, whatsoever.

### Comparison to other approaches

This table may help you understand how Flow-based Programming is similar or different from other approaches:

| Criteria            | Flow-based programming                                           | Dataflow programming                                             | Imperative programming                                                    |
| ------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Mindset             | Data flows through the program and operations transform the data | Data flows through the program and operations transform the data | Operations are executed on data in certain order under certain conditions |
| Program model       | A program is a graph of connected components                     | A program is a graph of connected components                     | A program is a sequence of instructions                                   |
| Model visualization | Data flow graph                                                  | Data flow graph                                                  | Control flow chart                                                        |
| Representation      | Visual (graphs) + Text (components)                              | Visual (typically), Text (sometimes)                             | Text                                                                      |
| Parallelism         | Implicit: all operations are parallel by default                 | Implicit or explicit, depends on framework                       | Explicit: parallelism needs to be programed                               |
| Control model       | Active: components control their lifecycle                       | Reactive: components are activated by external events            | Active: components control their lifecycle                                |

What makes Flow-based Programming different from other dataflow programming tools is:

- 2 engineering perspectives: Graph and Component. Components describe the behavior of data transformations. They are typically pieces of conventional code written as text files. Graphs define how data travels between Components. Graphs are typically visual or at least written in a declarative language.
- Any Graph is a Component itself and may include other Graphs. Nesting allows for decomposing the program into comprehensive parts. Think of being able to zoom into each component to see what's inside.
- Component interface consists of input and output Ports. At the same time, Ports are the primary means of interacting with the outside world from inside the Component.
- Components actively read the incoming data and decide how to react to it. This is different from reactive programming, where reaction functions are automatically fired on events by the runtime.
- Components run concurrently and share no state. But inside each component can be either stateful or stateless.
- The messages (IPs) are typically self-contained objects rather than just primitive values, which makes communication and synchronization more effective. Think of them as Data Transfer Objects rather than just scalar values.
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
> 
> - a questionnaire sent on Flow-based programming mailing list a few years ago
> - a questionnaire I posted in GoFlow repository on GitHub
> - my observations around the FBP community on the mailing list and Twitter during the last 10+ years
> 
> There is no strong statistical model behind it. But given enough motivation, such a research can be conducted. 

Originally introduced as a general-purpose programming paradigm, Flow-based programming has also been applied successfully to other domains such as:

- Business applications
- Web services

However, in these cases it is way harder to shift from traditional way of thinking to truly benefit from the shift to the Flow-based paradigm. E.g. it is typically just faster to write a massive web service with a conventional framework like [Spring](https://spring.io/) than with FBP. It is worth noting nevertheless, that [originally FBP was applied](http://www.jpaulmorrison.com/fbp/history.html) to transactional Banking software where it was used rather successfully for a good sequence of years.

## Problems it solves

> **IMO Warning**. Everything that you are going to read below is my personal opinion. I've been participating in the Flow-based Programming community for 10 years, built an academic framework, and a few commercial Flow-based systems that worked in production. All the statements in this and the following sections are based on my own ideas, observations, and fallacies. Trust it at your own risk.

When I started to learn Flow-based programming in 2010, it had a massive wow-effect on me and seemed to me like the ultimate paradigm I want to be using from then till the point when AI replaces programmers at work. That was naive, in multiple ways.

A decade later, I still think Flow-based Programming has several important advantages that the Software industry is missing. But these advantages are not the ones which I anticipated in the beginning. In this section I will share the modern Software Engineering problems (not Programming problems!) that Flow-based Programming successfully solves.

### Comprehensibility of Software systems

Humans absorb new information much quicker when it's supported by visuals. When looking at a software system for the first time, we understand a diagram much quicker than a few hundreds lines of code that it describes.

Most software projects start on a piece of paper or on a whiteboard because that's just easier to start reasoning about things.

With FBP, they continue living on a whiteboard. At any point of time you can see on a diagram how an application is structured and how it behaves.

### Evolution of Architecture vs. Code

In conventional software development cycle, diagrams get abandoned soon after the implementation starts. It's very hard to keep both diagrams and code up to date, especially over a long period of time. Even if the original team was pedantically updating the diagrams every time they made changes to the code, the next maintainers may not be so consistent with it.

There is an opposite approach when the diagrams are generated from code. With that approach you can get relevant diagrams at any time, but the resulting diagrams are complicated, because they are a byproduct of the implementation process, rather than a design guidance. E.g. an autogenerated Class diagram is very useful when studying the structure of the codebase. But when you need to drive high level architectural changes, you cannot just start with diagrams as they are autogenerated and you have to update the code first.

**In FBP, diagrams _are_ code**. Problem solved.

### Granularity of Dataflow systems

The main issue with dataflow approach is that it often gets too granular and low level. It is not uncommon in dataflow systems to have diagrams the size of a tennis-court with hundreds and hundreds of components on them. There are too many details on such diagrams.

With FPB, you write low level components in conventional textual programming languages. Those languages were created to describe behavior, so let the tool do its best job. And then encapsulate implementation details behind the component interface.

You can choose any level of granularity: from atomic operations to large subsystems represented as Components. The optimal choice usually sits somewhere in the middle. 

### Isolation of Dataflow languages

Some dataflow languages force you to implement everything either with the visual language itself, or with a low-level language that it is implemented in (e.g. C).

With FBP you don't need to reinvent all the wheels, you just use existing libraries and your favorite language inside the components. Cross-language and remote communication is encouraged by use of [FBP Protocol](https://flowbased.github.io/fbp-protocol/), meaning that your application can as well be written in different languages or distributed across different machines.

## Problems it does not solve

Just like any other thing, Flow-based programming is not a silver bullet. There are some problems people believe it to solve but in fact it doesn't or it does only partially.

### Designing clean software

It doesn't come for granted. Best described with a quote from [Henri Bergius](https://bergie.iki.fi):

> In FBP, your spaghetti code actually looks like spaghetti.

You still have to work hard and apply your best knowledge to design clean software with FBP.

The bad news is that many existing software patterns simply don't work when you change perspective from control flow to data flow. For example, if you try to implement MVC with complex Data Abstraction Layer glued by Dependency Injection, it is going to end up in a huge mess you stop understanding just 5 minutes after you create it. It is better to implement such things on top of the conventional frameworks.

There are few patterns which work effectively (e.g. [Assembly line](https://github.com/noflo/noflo-assembly/wiki/Introduction)), and there are more to be discovered by trial and error.

The good news is that it's way easier to spot when something is wrong with the architecture of a Flow-based system, because you can actually _see_ it.

### Building software faster

Programmers often seek a new programming paradigm because they believe it will make their life easier. Many FBP enthusiasts believed that FBP will make them more productive.

Some expected the components to be largely reusable (see "Having no code to write" below). Some thought that FBP components are less error prone (which is arguable) or at least easier to test (which is often true). Some believe in the power of visual debugging. Some believe in the power of visual UI versus typing text.

In reality the speed of building software depends on the following basic factors:

1. How good you know the tool

2. How many building blocks already exist

3. How much essential complexity is contained in the problem you solve

There is also the component of accidental complexity, but I would really like to emphasize the [essential complexity](https://en.wikipedia.org/wiki/Essential_complexity) part. If you don't like the fact that it was introduced for control flow rather than data flow, take [entropy](https://en.wikipedia.org/wiki/Entropy) of the system. The law of information science says: your program cannot be more simple than the problem it solves. A.k.a. [No Silver Bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet).

### Having no code to write

Another false FBP expectation is that the community can write such a library of reusable components, that later on there is very little code to be written. You'd drop the existing components into your graphs and that's it.

This assumption made people concentrate on writing components which are either too small or too generic. Which made neither of them easy to use. With very fine-grained components your graphs get too big and too convoluted. With very generic components you end up writing a lot of adapters and workarounds. So, when it comes to component scale, the golden mean works the best.

If your application is not just an example of a library usage, and rather solves some business or user problem, it means that you have to write a lot of components specifically for your application.

FBP can be used to build a [Low-code](https://en.wikipedia.org/wiki/Low-code_development_platform) or a [No-code](https://en.wikipedia.org/wiki/No-code_development_platform) development platform. One of the early attempts was e.g. [Flowhub](https://flowhub.io/). However, FBP is not alone in that niche today.

### Issues of parallel and distributed systems

Originally it was thought that as all Processes run in parallel, and there is single ownership rule (each Information Packet is only owned by one Process), all concurrency issues are solved inherently.

Some of them are: in FBP you design your program as a concurrent system from the very beginning, so you have to think how the data flows between parallel branches in the graph from day one.

But the coin has the other side: as all Processes may run in parallel, you need a lot of synchronization. Especially in long-running applications processing concurrent requests.

And there goes the the problem of distributed transactions. FBP hasn't escaped it automagically.

In other words, FBP makes parallel programming more visual, but all of its issues still have to be solved by a person who designs an FBP program.

## Can I use FBP today?

Yes, e.g.

- with Java: [JavaFBP](https://github.com/jpaulm/javafbp) + [DrawFBP](https://github.com/jpaulm/drawfbp)
- with JavaScript: [NoFlo](https://noflojs.org/) + [Flowhub](https://flowhub.io/)
- with Go: [GoFlow](https://github.com/trustmaster/goflow), [Flowbase](https://github.com/flowbase/flowbase), [Cascades](https://github.com/cascades-fbp/cascades)
- with Rust: [Fractalide](https://github.com/fractalide/fractalide)
- with PHP: [Railway-FBP](https://darkwood-fr.github.io/railway-fbp/)
- with Python: [multiple packages](https://wiki.python.org/moin/FlowBasedProgramming)
- probably with your other favorite language, Google knows how many Flow-based Programming runtimes are around.

Will it be easy? Maybe yes, but most probably not. Unfortunately we are not in a state when we can just throw all the UML and piles of classes away and switch to Graphs and Components all of a sudden. There are some important problems to be solved first. In particular, the learning curve may be steep both because of the paradigm shift and because of maturity of existing tools.

## Challenges of Flow-based Programming adoption

This section covers the problems faced by people who want to get started with Flow-based programming, as well as other factors that explain why FBP is not so popular yet. To be more constructive, I suggest some steps to overcome those issues.

### The barrier of entry

Getting started with Flow-based programming is not easy. From day one a new joiner has to overcome the following obstacles:

- Shifting a mental paradigm from classes, methods, loops and ifs to components and data streams
- Lack of documentation and examples of solving real world problems
- Complicated integration with existing software
- Lack of tooling, including a modern and convenient IDE

More on these problems and potential solutions below.

### The strive for a convenient IDE

Presently there are two big players in this market: [DrawFBP](https://github.com/jpaulm/drawfbp) and [Flowhub](https://flowhub.io/). And they are the very opposites of each other.

![DrawFBP UI example](/img/post/fbp/drawfbp_ui_example.png)

DrawFBP is a Java app that was started long ago and its interface isn't very appealing to programmers of the modern days. It uses very own UI flows to create FBP diagrams. It also comes with a code generator feature which works best for the classic JavaFBP. But it doesn't support universal `.fbp` or `.json` graph notations and isn't very useful for anything than just sketching (hello, UML) if you use a runtime that is not explicitly supported by this tool.

Flowhub (and its open source sister [Noflo-UI](https://github.com/noflo/noflo-ui)) is an app that runs right in your browser, and it's an IDE which does everything: edits graphs and components, saves projects on GitHub, runs tests, talks to the runtime, introspects the running application. And it does so by making use of [universal protocol](https://github.com/flowbased/fbp-protocol) and file formats. Flowhub had a lot of amazing features in the peak of its popularity around 2015, but it didn't really dominate the market. IMO, for the following reasons:

- The hosted version is not a full package: the IDE is in the cloud, but you need to set up and host the runtime part of your application manually
- If you run the NoFlo-UI yourself, it becomes too many parts that you need to manage, which slows down the development process
- The IDE is very smart, but when you need to do something out of ordinary, that is very likely unsupported and you have to hack it around
- Local debugging is complicated

I can see two directions that FBP IDEs can follow. And those are quite opposite.

#### A simple and convenient Graph Editor

Many of us FBP hackers are control freaks and fans of simple and tangible tools. We don't like heavyweight IDEs, especially if they are hosted and controlled by some 3rd party company renting  as a subscription.

What we need is neither Blender nor Eclipse. What we need is more like [Visual Studio Code](https://code.visualstudio.com/) these days. We want something that can edit `.fbp` and `.json` files in a visually appealing and intuitive way. And that's it for now. The rest can be done by firing up a command in CLI. We want to try things fast locally and see how they work/break. So if we had something that looks like Flowhub but edits local graph files, it would be just enough. If it would be just a plugin for VSCode, that's even better!

#### A cloud IDE

Some of the new comers, especially people attracted by Low-code and No-code concepts, prefer the IDE to do most of the work for them.

They need something like Flowhub but better: automatically hosting the application runtime somewhere in AWS, GCP, Azure, etc. Some people around Dataflow and Flow-based programming communities are actually working on such IDEs. But having experienced the development of Flowhub first hand, I can tell you that that's a very complex and expensive journey.

### Integration with existing software

FBP runtimes/libraries/frameworks should be easy to embed in existing projects. [NoFlo's asCallback](https://noflojs.org/documentation/embedding/) is a good example of such embedding feature.

Clean API, documentation, and examples are of undoubted importance. But programmers want to be able to start with what they already know and have.

When looking at business customers rather than individual hackers, integration into existing systems is even more important. Greenfield projects are much more uncommon rather than a bunch of existing services or a good old monolith. When bringing a new tool to the table, such as FBP, the expectation of a business is that it should add value rather than sacrifice what's already there.

### Discovering best design practices

There is a big mindset shift when switching from control flow thinking to data flow thinking. Difficulties don't end there though. Building something larger than a quick example application typically raises a lot of questions:

- How to split layers? Should I go for Model-View-Controller, or Data-Application-Presentation?

- Should there be layers at all?

- Package components by graphs or package components by function?

- Multiple inputs vs. single input

- Bulky everything-and-a-kitchen-sink messages vs. small objects

- How to pass state?

- Representing and controlling transactions

- How to reuse logic between components?

And many more. Trying to find an answer in a book on Object Oriented Design Patterns typically doesn't help, because most of those patterns and best practices were created with control flow in mind. Switching a paradigm brings a challenge of collecting best practices from scratch, piece by piece.

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

### Not invented here

One pattern that clearly exists in the Flow-based Programming community is called "Not invented here". Many enthusiasts who get excited and inspired by FBP set on a journey of creating their own FBP framework, IDE, or entire paradigm. There can be multiple reasons for that, e.g.:

- Exploring the concept and playing with code to understand it better

- Building something very niche to what the person is working on

- Having a strong personal vision of how to "get it right"

- Thinking that the prize is going to be huge, and not willing to share the glory

No matter what the reasons are, the fact is: very little collaboration happens in the community. Which is very unfortunate, because it takes a lot of effort and typically a lot of collective effort to get a technology to a state where it can grow its popularity organically.

Frankly speaking, if you ask me "why hasn't Flow-based Programming become popular in the last 20 years?" my answer will be: because everybody was too busy building their own versions of it. Changing this attitude is a must for FBP to grow from striving to thriving.

## Beyond Flow-based Programming

In this chapter I think out loud about what is missing in the original Flow-based Programming that modern Software Engineering needs, and where the whole thing may be going.

### Microservice orchestration

One of the interesting use cases that FBP may have is providing a higher level view on building microservice systems. Microservices can be small enough to be considered as Components, and message bus can be used as a transport layer for Connections. Value that FBP adds on top of what is already there is:

- Visualizing system architecture at higher level, with the picture being always up to date

- Standardizing the interface and communication between services

- Providing real-time introspection of cross-service communication (if implemented)

This is the direction where e.g. [Cascades FBP](http://cascades-fbp.github.io/) was going.

### Stream processing

Many large organizations nowadays embrace the use of stream processing frameworks, such as:

- [Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)

- [Apache Flink](https://flink.apache.org/)

- 
