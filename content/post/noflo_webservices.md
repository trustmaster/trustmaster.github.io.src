+++
date = "2015-01-23T14:38:41+01:00"
title = "Running web services as NoFlo graphs"
tags = ["noflo","fbp","development"]
comments = true
draft = false
description = "Some problems that occur when trying to run entire web services as NoFlo graphs and some (radical) ways to solve them"
image = "/img/post-bg.jpg"
categories = ["Flow-based Programming"]

+++

# Getting All Things NoFlo

This article describes things which are yet TODO to get the entire server architecture running on NoFlo (and in Flowhub), problems related to them and potential solutions.

![NoFlo all the things!](/img/post/noflo_all_the_things.png)

## Request Isolation

By default there is no isolation between IPs belonging to different user requests in NoFlo networks.

### Solution 1 (current): a new Network instance per request

Express does its job conventionally, while request handlers instantiate NoFlo networks on demand.

**Pros**:

* Easy to implement
* Good level of isolation without much effort

**Cons**:

* Doesn’t let us run the whole app as a NoFlo network
* Waste of resources

**Conclusion**:

* We need to move forward from it!

### Solution 2: group-based synchronization

Use the outmost (first in, last out) NoFlo group packet as request identifier. The entire app is a NoFlo graph, requests are born in noflo-xpress components and tagged with UUIDs, then all downstream components need to handle the outmost group carefully to synchronize data belonging to the same request.

**Pros**:

* Possible with current NoFlo and WirePattern
* Doesn’t hide all the magic under the hood, you see what makes synchronization happen and can override it

**Cons**:

* If any component of any subnet omits the outmost group or changes group order -> crash! boom! bang!
* Error handling is tricky (see chapter below)
* You still need to keep synchronization details in mind

#### Use cases for groups

Summarizing here what groups have been (reasonably) used for so far in NoFlo:

1. Passing request/job IDs the data belongs to.
2. Passing XML-like data structures, splitting them into parts and composing back.
3. Grouping UI events (<graph><event>data</event></graph>).
4. Tagging data with its source URL.

#### Groups in component libraries

Most of component libraries were written in NoFlo 0.3 and 0.4 days. Most of them disrespect groups and some of them even have synchronization bugs (e.g. core/Kick is race-prone in some cases).

Our original plan about it was to run a campaign porting all the libraries to 0.5 and using WirePattern when necessary.

With group forwarding required in every single component and with synchronization required in every 3rd or 5th, this is going to lead into a huge WirePattern spaghetti dish. The average NoFlo component won’t look elegant anymore.

My proposal consists of the following:

* Build WirePattern into noflo.Component instead of noflo.Helpers.
* Use port attributes to define some of the synchronization properties.
* Enable group forwarding by default.
* Introduce one or multiple "process" functions per component. Each function has its own “firing pattern” (a constraint on inport events that triggers the function).
* Release these changes in NoFlo 0.6 and then run the library updating hackathon.

### Solution 3: request tokens as 1st class term

Application architecture is similar to Solution 2, but implementation is different. Instead of using outmost groups, add a concept of optional "request identifiers" or “context tokens” to NoFlo itself. Once a context token is attached to a substream, it sticks to each of its IP. Every component isolates data (input, output and internal state) using context tokens as scope keys and forwards context information downstream automatically. By default no context keys are used, so all data is in the same scope just like they didn’t exist. Context tokens are only used in applications which require request isolation.

*Footnote: contexts are mentioned in Matt Carkci’s Dataflow book, so this idea is not just crazy, it’s known to work.*

**Pros:**

* Fixes the group consistency problem of Solution 2.
* This would require minimal to no changes to existing components and subnets as most of context handling is done by NoFlo runtime itself.

**Cons:**

* One more keyword in NoFlo language, and yet it’s not in "classical FBP" language either.
* Despite keeping the current NoFlo 0.5 API, it requires heavy changes to WirePattern and other parts of the runtime.

## Error handling and Synchronization

Correct error tracking is tricky in a divergent-convergent graph because an error may happen at any node while other parts of the request may be hanging in other parts of the graph. They have to be pulled together somehow and forwarded back to the client properly.

To make things more complicated, there are different kinds of errors which need to be handled differently:

* Warnings (e.g., input validation errors): it is better to check for as many of these as possible and collect them in a list before sending a response to user.
* Errors (e.g. database record not found, third party service time-out): such errors stop further processing of the request immediately with an error message, but they are considered as a normal part of the application flow.
* Fatals (e.g. undefined variable or database connection is down): such errors not only stop further processing of the request but also raise a kind of notification to a developer that something is going really wrong and most probably human’s attention is needed.

Handling these errors is most likely out of scope of NoFlo itself, but is rather a matter of architecture patterns.

### Solution 1: "String of pearls"

J. Paul Morrison suggests using "string of pearls" FBP pattern in all applications dealing with requests. In this pattern, all the data for the request is collected within a single substream. The substream floats from one pearl in the string to another (just like on a pipeline). Each pearl on a string may pass the substream to its side processes (e.g. to load missing data, etc.) before forwarding it to the next pearl. Side processes return substreams back to pearls. At the same moment of time the request (and all the data associated with it) belongs to a single process, so if an error occurs, the current pearl just finishes the request without having to gather it from a branchy graph.

**Pros:**

* Reliability of this pattern has been proved by time and real banking applications. (and JPM :-))

**Cons:**

* It encourages concurrent processing of different requests but restricts concurrent processing of the same request. While in some applications parallel tasks would really reduce response time and make users happier.

### Solution 2: Request processors

This pattern is very similar to "String of pearls" but allows a certain level of divergence-convergence to do parallel processing.

It introduces Request processors - a special type of processes which own the requests, scatter data to subgraphs and gather them back for a response.

Request processors make up a "backbone" of the application. Similar to string of pearls, requests float down the backbone of processors. But when a processor needs some side work to be done for a request, it doesn’t pass it with all the data to a single side process, but rather holds the request and sends the data to its workers with request ID attached. The workers do their job and return the results (and/or errors) to their master. The master then gathers the results and decides either to pass the request to the next processor in the backbone, or to finish it with a response immediately.

**Pros:**

* Maintains request consistency just like "string of pearls".
* Allows parallel computations for the same request.

**Cons:**

* Synchronization code of Request processors is reusable, but its implementation is non-trivial.#
* Caution: thin ice! We don’t know how it will behave in real world.

## Programming, Motherfucker

OK, we are done with theory and fundamental problems. All the rest is just [http://programming-motherfucker.com](http://programming-motherfucker.com)

1. Implement component libraries (e.g., forms, templates, DB queries, etc.)
2. Wrap everything in components
3. Connect components into graphs
4. Convert old graphs from FBP to JSON
5. Make sure it runs in Flowhub
