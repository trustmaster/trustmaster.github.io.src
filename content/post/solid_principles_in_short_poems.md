---
categories: ["Development"]
comments: true
date: "2018-03-12T22:33:45+01:00"
description: "An entertaining summary of SOLID design principles, with common sense shortcuts and simple poetry"
draft: false
image: "/img/post-bg.jpg"
tags: ["architecture"]
title: "SOLID design principles in short poems"
---

Software Design is hard. It is complicated, it takes years to learn (in a sense that you really know what you are doing and why, compared to just copying and pasting something that you read somewhere) and sometimes it is counter-intuitive.

I started making these notes to better understand SOLID design principles myself while reading the ["Clean Architecture" by Robert C. Martin](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164). The book points to common misunderstandings of these principles, but the explanations given are far from being easily memorable.

To make Software Design a bit more entertaining, I decided to write a short funny poem for each of the five SOLID principles, complemented with examples from real life that ideally even non-engineers can understand.

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

So far so good: the `Printer` has only one method `print()` which does something relatively simple (i.e. just prints whatever it's sent). But there are three actors depending on it (no matter if they inherit it, call it directly or call it via shared `interface`). Thus, it violates SRP. Indeed, let's say that the `DesignDepartment` is doing well in graphical design and now they need a much better `Printer` to print full-color A0-sized posters on glossy paper. What do you do? Add another method `printA0()` or make the `print()` method print full-color A0s by default? `TechDepartment` is amused by the idea of using that printer to make wall-sized comics! But `FinanceDepartment` is angry, because the geeks would be wasting the ink, the time of everybody waiting in the printer queue, and of course company $$$s. Thus, the obvious solution is to have not 1 but 3 different Printers, each one fitted for the needs its actor:

![SRP in action](/img/post/solid/srp_example_2.svg)

That brings a temptation of explaining SRP as "Each class should have only one class dependent on it", but in reality this would mean tons of merely duplicated code:

To deduplicate our effort we could make a reasonable violation to the SRP principle. For example, `FinanceDepartment` and `TechDepartment` could still use printers of the same cheap model, while `DesignDepartment` would be using its shiny `GraphicPrinter`:

 ![Optimized printer diagram](/img/post/solid/srp_example_3.svg)

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

As an example, we'll use a famous electronics company which has been producing a very successful model of a `VacuumCleaner` for many years. Its `VacuumCleanerBase` has a powerful and reliable `Engine`, all its components have been perfectly fit together and proved over time. The time though keeps going by and the governments come up with energy consumption regulations, so the mighty but fixed `1800 Watt` power characteristic is no longer an advantage. What does the company do? Invent a new model, spending lots of money for research, engineering, testing, etc.? Modify the internals of `VacuumCleanerBase` so that it is no longer well-fit and reliable? No, there's a better way. The old good `VacuumCleanerBase` can be extended using a simple `PowerInvertor` device that will allow a user to adjust the output wattage using a simple control knob or slider:

![Extended VacuumCleaner](/img/post/solid/ocp_example_1.svg)

Of course, in order to support such an extension, `VacuumCleanerBase` should already provide all the necessary interfaces. This kind of extensibility is one of the requirements of OCP.

Now, the company's Marketing department makes a research and finds out that there is a huge demand on `AntiAllergicVacuumCleaner` model from people having pets, rugs or living in dusty environment. Here is what the Engineering department thinks of it: easy! All they need to do is to design a `TurboBrush` for the rugs and replace standard filters with a high class `HEPAFilter`:

![Anti-allergic VacuumCleaner](/img/post/solid/ocp_example_2.svg)

For an electronics company this approach saves millions of dollars on engineering, testing and modifying production lines and supply chains. In software engineering, it can save quite some human hours at least.

Rephrasing the principle:

*A component should be so simple that it rarely needs to be modified, but it should be easily extensible by other components.*

## Liskov Substitution Principle (LSP)

```
A founder founded a foundation
Of large software corporation.
Foundation was so strong and hard,
It found a founder's counterpart.
```

The original definition is:

> If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T.

This definition is cryptographically scientific, but the easier the example we need. Let's pick a slightly nostalgic one, with phone chargers and data cables.

Not so long ago, a hardware designer who was developing, say, a new Nokia or Sony Ericsson cell phone, would fit it with an A/C port, that a customer can plug a charger in. In the beginning, each model had its own type of charger. Then engineers (or was it marketing?) realized, that they can actually unify the chargers, so that they are _interchangeable_ for all the cell phones produced by the company. Your Nokia is running flat but your colleague has a Nokia too? You can borrow their charger, what can be more convenient:

![Unified Nokia interfaces](/img/post/solid/lsp_example_1.svg)

In this case your colleague's charger is the _substitute_ of the charger that you left at home. But then the principle got further: before you used a separate charger port and data cable port, but then some smart people came to use the same micro-USB for charging the phone and sending the data in and out. Just mounting the port on a device is not enough, the engineers had to ensure that all the peers would follow the same protocol, i.e. use the same _interface_. An absolutely wonderful consequence of following the LSP principle further this way was the fact, that now lots of phones of different brands could use the same chargers and the same data exchange software:

![Unified micro-USB interfaces](/img/post/solid/lsp_example_2.svg)

This diagram is simplified to encapsulate more components and implementation details, but what's important is that the combination of the following _interfaces_:

- _micro-USB Port_
- _micro-USB Cable_
- _MTP Data transfer drivers_

makes it possible that we can interchange any phones, any cables and any computers that implement these interfaces and they will just work together.

Were there exceptions? Yes, for instance Apple devices kept using proprietary chargers. And even nowadays some devices, despite using the common micro-USB or USB-C ports, require brand-specific software instead of using the common protocols, giving us a good example of LSP violation. With violation of LSP comes the limitation of possible use cases, extensibility and convenience.

Back to software engineering theory, the principle can be rephrased as:

*Use interfaces to make the dependency components interchangeable.*

<small>_Note: the original definition doesn't say anything about interfaces. It sounds more like using parent classes instead of child classes. That is so C++. This article targets slightly different languages and encourages using interfaces and aggregation over inheritance._</small>

## Interface Segregation Principle (ISP)

```
Man proposed to a princess is no fool:
He's given a horse and a kingdom to rule.
But there's a dragon to slay,
And an army to pay,
While all he wants is a girl and a mule.
```

A quote from Robert Martin's book can serve as a definition:

> it is harmful to depend on modules that contain more than you need

Although the name of this principle sounds somewhat chemistry, we can fall back to a simple example that you may have experienced in real life. Imagine that you just moved to a new home and you demand on broadband `Internet` access. There are several Internet Service Providers that can connect your apartment, but only one of them can do it right away while with the others you have to wait for several weeks.

That provider though only offers ADSL `Internet` that comes with a `Landline` and a `MobileSIM` package, all paid on a monthly basis with a minimal contract of 2 years. You don't want to wait several weeks and decide that having a landline and a relatively cheap SIM is not such a bad thing. So, you sign the contract and start to depend on the following congregated interface:

![Depending on a multi-purpose interface](/img/post/solid/isp_example_1.svg)

_This diagram shows just a few of the functions that you start to depend on as a daily routine, provided from an unsegregated <s>interface</s> Internet provider._

After the first month you come to a conclusion that even though the ADSL is more or less OK, the calls on that SIM are pricey and you don't need the landline at all. Altogether you've signed to pay twice as much for the services that you don't need and there are way better deals on the market if you got a broadband and a SIM card separately:

![Depending on almost segregated interfaces](/img/post/solid/isp_example_2.svg)

In the latter case if the ADSL line goes nuts, you can change it for a better provider, without having to change your mobile SIMs as well.

So, what this principle recommends us is:

*Don't depend on things that you need and that you don't need altogether. It's better if they break separately rather than all at once.*

## Dependency Inversion Principle (DIP)

```
There lived a tail who waged a dog.
It could not stay like that for long.
They went to lawyer to conclude
Their independent neighborhood.
And now there's nothing left to nag:
They both have documents to wag.
```

As stated by Robert C. Martin:

> A. High-level modules should not depend on low-level modules. Both should depend on abstractions.

> B. Abstractions should not depend on details. Details should depend on abstractions.

This principle is the most difficult one to explain and illustrate to somebody who hasn't spent countless hours building software. Dependency what? Inversion how? For sake of who?

But thinking of it, this principle is widely involved in how we learn and interact with the outer world: we interact with concrete objects via abstract interfaces. And these interfaces serve as protocols, that allow the dependency flow to be the inverse of the control flow, at least in the critical parts.

For example, say that you need a plumber to fix a leak. Alright, you know your friend George is good at things and probably can do it for you. Just ask George then. But what if George is not around or he doesn't know how to fix a leaking tap in particular? Then you need an abstraction: a Plumber. And you need another abstraction to get you that Plumber in first place: some Household Service or even just Classifieds (both are just Abstract Factories in this case).

Let's zoom in and generalize this example a little bit. So, consider we have a `Customer` who relies on plumber `George` entirely:

![A very concrete dependency](/img/post/solid/dip_example_1.svg)

It's good to know there's `George` to fix a leaky pipe. But what happens if `George` is not there when a `Customer` needs him? Or if the problem a `Customer` has later on requires skills or equipment `George` simply doesn't have. It's not so good for `George` either if the only way he gets customers is via direct reach. Thus, this direct dependency may be effective in short term but not flexible over time.

With Dependency Inversion Principle applied, the diagram would look something like this:

![DIP in action](/img/post/solid/dip_example_2.svg)

In this case, a customer called `Bill` implements an abstract interface `Customer` and has abstract knowledge on how to get help from a generic `HouseholdService`, e.g. when he needs a `Plumber`. The interface `Plumber` represents what a `Customer` thinks a plumber should do. All the rest is behind the scenes: a double curved line also known as architectural boundary. For example, there can be a handy website `FindAPlumberDotCom` that helped `Bill` to find a plumber `George` within seconds without any hassle.

So, where is the inversion? When `Bill` has a leaky pipe, he still calls `George` to fix it in the end - this is the control flow. But `Bill` doesn't depend on `George`, he depends on an abstract interface that `George` also depends on. A dependency from `George` to `Plumber` interface is inverse to the control flow. This is why the principle is called Dependency Inversion.

Back in Software Engineering world, DIP means _applying abstractions to implement achitectural boundaries and make the dependency flow in the direction stated by the principle: from concrete details to stable abstractions_.
