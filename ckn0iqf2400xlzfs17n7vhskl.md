---
title: "Clean Architecture Applied"
datePublished: Mon Apr 15 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqf2400xlzfs17n7vhskl
slug: clean-architecture-applied-review
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617801226272/3qlQHi4jd.jpeg
tags: csharp, software-architecture, clean-code

---


During development, I already noticed some benefits and drawbacks.

Let's start with the drawback: when I change a data structure, such as the `Customer`, there's suddenly a few locations where this needs to be changed such as the tests and the repository. This isn't unexpected, but I noticed there are a lot more changes than in other code. This can be because this experiment made me more aware or because the architecture accentuates this. In hindsight it is not such a big disadvantage, but more an observation of how this architecture brings these things to light.

A general observation about Clean Architecture is that this architecture is overkill for small project. It offers a lot of benefits, but I need to put in a lot of effort to get those benefits. Without all the interfaces and separate projects, I could be done much faster. Yet that would mean that it would be a lot less extendable. The extensibility can be added afterwards when I would need it. For example, I could put an `IWorkRepository` interface in place when I need to add additional data sources. It's always a push and pull between [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) and not enough flexibility that make changing the design later difficult. The added interfaces and extension points did make me think twice about how I would go about some part of the code, which did benefit the end result.

The biggest advantage of this architecture is that it's easy to extend. That's not a big surprise as this is one of the pillars of Clean Architecture. Changing the single implementation of `DinkToPdfDocumentGenerator` into a wrapper for the internal functions that became `FluidHtmlDocumentGenerator` was effortless. There have been other places where this became apparent, but this was one of the most obvious.

An extension of the previous advantage is that the application is nicely compartmentalised. This allows me to focus on one problem at a time. It also forces me to keep my components small as they only focus on one part of the system. Small components are easy to reason with and the logic not only fits on a screen, but in my mind as well. I understand the problem space a lot better when it fits in my head and it helps me find a good solution for the problem at hand.

To be sure that components are truly separated, an inner circle component should never use the classes of the outer components. For example, in the `TimeCampWorkRepository` I have an object that allows me to deserialise the JSON structure I get back from the TimeCamp API. That object is not getting passed to the upper structure. I have a more generic object I map to so my inner components do not have to rely on the specific TimeCamp structure.

The `WorkDay` object is more than just a simplified view of the TimeCamp api. It's a contract that describes what the component needs to do it's work properly. If I use different repositories, for testing for example, all I have to do is make sure I return a good `WorkDay` object and my code will run.

Lastly, testing is not exclusive to Clean Architecture, but I can't ignore their importance: tests really saved me a lot of time. While experimenting with certain alternatives or ways how to solve a problem, the tests showed me very quickly if I was going the right way. I use a mix of black box testing where I compare the result of a generation with a predetermined document and behavioural testing to check that I call some external services correctly.

#### The end

Clean Architecture is nothing new in my opinion, but it is a good set of principles to build an application upon. Don't expect any ground breaking insights from the book, but a confirmation of what we, as a profession, are trying to work towards in terms of a good foundation. It offers a set of principles that outline how to structure an application so that it's easy to build, maintained and extended.

I, for one, will be using this template as a starting point whenever I need to build a new application or when I need to work towards a better structure in legacy applications.
