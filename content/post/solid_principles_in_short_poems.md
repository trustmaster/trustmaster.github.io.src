+++
categories = ["Software Design"]
comments = true
date = "2017-11-25T16:33:45+01:00"
description = "An opinionated summary of SOLID design principles, which common sense shortcuts, a grain of Zen and simple poetry"
draft = true
image = "/img/post-bg.jpg"
tags = ["design", "architecture", "patterns"]
title = "SOLID design principles in short poems"

+++

# SOLID design as poetry

Disclaimer: I make these notes to better understand SOLID design principles myself while reading the "Clean Architecture" by Robert Martin. The book points to common misunderstandings of these principles, but the explanations given are far from being easily memorable.

Let's take a deep breath and on an exhale give a clean explanation to each of the principles from scratch. With a bit of poetry and fun.

## Solid Responsibility Principle (SRP)

```
A servant serves a single stew
To eaten be by only you.
Whenever master's mind is changed,
The stew is promptly rearranged.
```

This principle is usually (mis)understood as _"A class should only do one thing"_ or even _"A class should only have one public method"_. But the actual definition is:

> A module should be responsible to one, and only one, actor.

The difference is easy to illustrate with an example. Consider a software model of a small `DigitalAgency`. It has 3 departments: `FinanceDepartment`, `DesignDepartment` and `TechDepartment`. And there is one `Printer` that serves all the needs in this small company: Finance employees print their reports, graphical Designers print previews of their work, and the Tech geeks print their online bookings, xkcd comics and what not:

![Example class diagram](/img/post/solid/srp_example.svg)

So far so good: the `Printer` has only one method which does something relatively simple (i.e. just prints whatever it's sent). But there are three actors depending on it (no matter if they inherit it, call it directly or call it via shared `interface`). Thus, it violates SRP. Indeed, let's say that the `DesignDepartment` is doing well in graphical design and now they need a much better `Printer` to print full-color A0-sized posters on glossy paper. `TechDepartment` is amused by the idea of using that printer to make wall-sized comics. But `FinanceDepartment` is angry at the idea of geeks wasting the ink, the time of everybody waiting in the printer queue and of course company $$$s. Thus, the obvious solution is to but 3 different Printers, each one fitted for the needs its actor.

That brings a temptation of explaining SRP as "Each class should have only one class dependent on it", but in reality this would mean tons of merely duplicated code. Instead, we could group those dependents by a combination of two factors: _expected interface_ and _expected behavior_.

In simple words:

**There should be only one way and purpose of using a component**

Or in a more lengthy form:

**There should be only one dependent of a component, or all the dependents should share the same expectation of both the component's interface and behavior**

I tend to think that the poem given above is a simple and vivid illustration to this principle.

## Open-Closed Principle (OCP)

```
The book is open, but it's closed.
It's read, adopted and exposed,
But to add more ideas
Without legal fears,
A new one has to be composed.
```

The original definition reads as:

> A software artifact should be open for extension but closed for modification.


