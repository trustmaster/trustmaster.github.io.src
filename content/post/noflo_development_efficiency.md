+++
date = "2016-11-07T17:39:26+01:00"
title = "Development efficiency challenges in NoFlo"
comments = true
draft = false
description = "Obstacles on the way to happy day-to-day life with NoFlo and ways to overcome them"
image = "/img/post-bg.jpg"
categories = ["NoFlo","Flow-based programming"]
tags = ["noflo","fbp","development"]
+++

# Development efficiency challenges in NoFlo

Programming with NoFlo is getting more and more smooth with every release. However, if maintaining a NoFlo application is your day-to-day job you may notice some inconveniences.

## Defining new components is slow

Every time you add a new component you have to define a pretty massive hash map of its ports, their data types and preferably at least descriptions. This is cool as it encourages you to declare as many details as possible but it’s not very friendly to rapid prototyping where you want just to list the ports and get your hands on the code and add details later.

### Solutions

* Reduce minimal boilerplate for new components

## Wiring up all the ports is slow (.FBP)

OK, maybe it wouldn’t be so slow if it was all done in Flowhub but until those glory days when we could switch all our projects to Flowhub, the problem persists.

Components have many ports. Say there are 8 components in the graph, each of them has 4 ports and there are also 2 inports and 2 outports in the graph. This could make at least 32 connections required for the graph to run. Which is writing 32 lines of code in .FBP, and what’s worse keeping them all and the whole picture in mind. Using [NoFlo visualize tool](https://noflojs.org/visualize/) helps a lot here though. But then you close that graph, open it 3 months later to add a new branch and rewire some connection and you can’t do it without a migraine or two.

With Flowhub it’s a bit easier b/c you see the picture, attach and detach connections and rewire it more or less effectively.

### Solutions

* Reduce the amount of ports and connections in the graphs
* Use more terse syntax than .FBP
* Switch to Flowhub eventually

## Can't tell if a graph works until everything is wired

It’s easier to code something in small iterations one piece at a time: add a component, add tests for it, add another component with tests, wire these two components, make sure they work together, add a third component, etc. then you have the whole graph and tests for it. This requires a lot of rewiring and intermediate tests.

As described above, in .FBP rewiring is expensive. In Flowhub rewiring is cheaper but still requires quite some clicks and drags, but what’s more important Flowhub/NoFlo won’t let you run the program until the graph is all wired up correctly. So what we do in practice is not trying anything until the entire graph is complete with all the components and connections. Which is a lot of headache wiring first (in .FBP) and then lots of bugs when it’s finally wired (both .FBP and Flowhub).

### Solutions

* Do write tests for components and subgraphs
* Prefer mid-sized components and subgraphs so that each of them can have a test without too much hassle

## Doesn't show which component failed/stuck

This is the hardest thing when debugging. So you know that the graph just hangs somewhere, at some component. But you don’t know which one. There is [Flowtrace](https://www.npmjs.com/package/flowtrace) at your service but reading a flowtrace of a real application is hell of a job, there can be hundreds of socket events and you have to figure out which ones are actually *missing* from those hundreds. Replaying a flowtrace in Flowhub with `flowtrace-replay` is slightly better but the problem persists: you need to find the missing piece of jigsaw.

A less critical but also annoying thing is that when something fails somewhere, you see the error in the output and it maybe even contains a stack trace to some extent but it doesn’t contain the process or component name, which you’d like to know in first place when opening an error output.

### Solutions

* Always include component names in stack traces
* Always catch and log errors
* Visual flowtraces in Flowhub
* How could we fix the missing jigsaw problem?

## Conditionals/branching is hard

There are no simple IFs in FBP. No such thing. If you need to check something, it’s either inside a component or a whole branch of the graph. Usually if you need an IF in the graph, it results into 2 graphs. Which takes time wiring up. And duplicates the code. And duplicates the bugs.

"Stop thinking in control flow you idiots, you should think data flow here!" - says the Captain FBP. Yes, sir, how do I think such things in data flow, sir?

### Solutions

* Simple graphs are easier to add conditionals to
* If you need to duplicate a branch, put it in a subgraph
* Keep control logic inside components

## Components flourish with promises/callbacks

Look inside a component that does some real job and you find yourself in the land of wild and unharnessed async JS. Callback hell - yes, promises - yes, `async/await` inside the components - oh yes, coming soon. Dataflow you say? Hardly so.

I suspect that one of the initial syncretic intentions of NoFlo was to split these vertical callback staircases and promise stacks into good old black boxes with wires between them. However, splitting every async call and callback into 2 components is rather expensive, so in practice we rarely do that.

But could NoFlo help us simplify async code so we don’t have to write so many callbacks or promises? Yes, partially.

### Solutions

* Thoughtful breakdown of the applications into components
* Use promises inside components

## Coffee is cool but it’s ES2015+ for most people now

CoffeeScript is really cool, it’s write less do more. Some people complain that all the cool kids are using ES6/7+Babel now and that CoffeeScript is completely abandoned. But it’s simply mature and cool and you still write less and do more with Coffee than with ES6+.

However, on a historical scale, CoffeeScript is most probably a stale creek. It’s popularity is not going to grow because of the cool kids and everybody busy watching the changes in ES spec. That means, if NoFlo wants to grow its audience, it has to embrace ES2015+ support. No-no, I don’t mean it should be rewritten in *Script (`s/*/(Type/Pure/whatever/`), I mean the ecosystem should be very friendly to people coming from ES world. The documents should contain ES examples first and there should be good quality component libraries in ES to be used as reference too.

### Solutions

* Documentation should be ES-first, CS-second
* Examples and reference libraries in ES

## What's next?

I've been addressing these problems and a few more while maintaining backends in NoFlo. Just like I created `WirePattern` back in the days of `noflo@0.5.*` (which is going for good with `noflo@0.8.*` and the [Process API](http://bergie.iki.fi/blog/noflo-process-api/)), I've created [NoFlo Assembly Line](https://github.com/trustmaster/noflo-assembly) which is a combination of design patterns, best practices and a ES6 library that works on top of NoFlo. It's not yet fully released and I plan to do so after NoFlo `0.8.0` is officially out.
