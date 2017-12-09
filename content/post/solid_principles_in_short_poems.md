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

Here I'll try to make them easier to understand and memorize, with a bit of poetry, fun examples and a grain of salt.

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

So far so good: the `Printer` has only one method `print()` which does something relatively simple (i.e. just prints whatever it's sent). But there are three actors depending on it (no matter if they inherit it, call it directly or call it via shared `interface`). Thus, it violates SRP. Indeed, let's say that the `DesignDepartment` is doing well in graphical design and now they need a much better `Printer` to print full-color A0-sized posters on glossy paper. What do you do? Add another method `printA0()` or make the `print()` method print full-color A0s by default? `TechDepartment` is amused by the idea of using that printer to make wall-sized comics! But `FinanceDepartment` is angry, because the geeks would be wasting the ink, the time of everybody waiting in the printer queue, and of course company $$$s. Thus, the obvious solution is to have not 1 but 3 different Printers, each one fitted for the needs its actor.

That brings a temptation of explaining SRP as "Each class should have only one class dependent on it", but in reality this would mean tons of merely duplicated code. For instance, `FinanceDepartment` and `TechDepartment` could still use printers of the same cheap model, while `DesignDepartment` would be using its shiny `GraphicPrinter`. So, we end up with a diagram like this:

![Updated class diagram](/img/post/solid/srp_example_2.svg)

In simple words, the SRP principle can be rephrased as:

*A component reports to only one boss.*

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

As an example, we'll use a famous electronics company which has been producing a very successful model of a `VacuumCleaner` for many years. Its `VacuumCleanerBase` has a powerful and reliable `Engine`, all its components have been perfectly fit together and proved over time. The time though keeps going by and the governments come up with energy consumption regulations, so the mighty but fixed `1800 Watt` characteristic is no longer an advantage. What does the company do? Invent a new model, spending lots of money for research, engineering, testing, etc.? Modify the internals of `VacuumCleanerBase` so that it is no longer well-fit and reliable? No, there's a better way. The old good `VacuumCleanerBase` can be extended using a simple `PowerInvertor` device that will allow a user to adjust the output wattage using a simple control knob or slider:

![Extended VacuumCleaner](/img/post/solid/ocp_example_1.svg)

Of course, in order to support such an extension, `VacuumCleanerBase` should already provide all the necessary interfaces. This kind of extensibility is one of the requirements of OCP.

Now, the company's Marketing department makes a research and finds out that there is a huge demand on `AntiAllergicVacuumCleaner` model from people having pets, rugs or living in dusty environment. Here is what the Engineering department thinks of it: easy! All they need to do is to design a `TurboBrush` for the rugs and replace standard filters with a high class `HEPAFilter`:

![Anti-allergic VacuumCleaner](/img/post/solid/ocp_example_2.svg)

For an electronics company this approach saves millions of dollars on engineering, testing and modifying production lines and supply chains. In software engineering, it can save quite some human hours at least.

Rephrasing the principle:

*A component should be so simple that it rarely needs to be modified, but it should be easily extensible by other components.*
